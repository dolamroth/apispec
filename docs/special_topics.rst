Special Topics
==============

Solutions to specific problems are documented here.


Adding Additional Fields To Schema Objects
------------------------------------------

To add additional fields (e.g. ``"discriminator"``) to Schema objects generated from `spec.components.schema <apispec.core.Components.schema>` , pass them
to the ``component`` parameter. If your'e using ``MarshmallowPlugin``, the ``component`` properties will get merged with the autogenerated properties.

.. code-block:: python

    properties = {
        "id": {"type": "integer", "format": "int64"},
        "name": {"type": "string", "example": "doggie"},
    }

    spec.components.schema("Pet", component={"discriminator": "petType"}, schema=PetSchema)


.. note::
    Be careful about the input that you pass to ``component``. ``apispec`` will not guarantee that the passed fields are valid against the OpenAPI spec.

Rendering to YAML or JSON
-------------------------

YAML
++++

.. code-block:: python

    spec.to_yaml()


.. note::
    `to_yaml <apispec.APISpec.to_yaml>` requires `PyYAML` to be installed. You can install
    apispec with YAML support using: ::

        pip install 'apispec[yaml]'


JSON
++++

.. code-block:: python

    import json

    json.dumps(spec.to_dict())

Documenting Top-level Components
--------------------------------

The ``APISpec`` object contains helpers to add top-level components:

.. list-table:: 
   :header-rows: 1

   * - Component type
     - Helper method
     - OpenAPI version
   * - Schema (f.k.a. "definition" in OAS v2)
     - `spec.components.schema <apispec.core.Components.schema>`
     - 2, 3
   * - Parameter
     - `spec.components.parameter <apispec.core.Components.parameter>`
     - 2, 3
   * - Reponse
     - `spec.components.response <apispec.core.Components.response>`
     - 2, 3
   * - Header
     - `spec.components.response <apispec.core.Components.header>`
     - 3
   * - Example
     - `spec.components.response <apispec.core.Components.example>`
     - 3
   * - Security scheme
     - `spec.components.response <apispec.core.Components.security_scheme>`
     - 2, 3

Most component registration methods provide a ``lazy`` keyword argument,
allowing to define a component but only publish it in the generated
documentation if it is actually referenced.

To add other top-level objects, pass them to the ``APISpec`` as keyword arguments.

Here is an example that includes a `Server Object <https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md#serverObject>`_.

.. code-block:: python

    import yaml
    from apispec import APISpec
    from apispec.ext.marshmallow import MarshmallowPlugin
    from apispec.utils import validate_spec

    OPENAPI_SPEC = """
    openapi: 3.0.2
    info:
      description: Server API document
      title: Server API
      version: 1.0.0
    servers:
    - url: http://localhost:{port}/
      description: The development API server
      variables:
        port:
          enum:
          - '3000'
          - '8888'
          default: '3000'
    """

    settings = yaml.safe_load(OPENAPI_SPEC)
    # retrieve  title, version, and openapi version
    title = settings["info"].pop("title")
    spec_version = settings["info"].pop("version")
    openapi_version = settings.pop("openapi")

    spec = APISpec(
        title=title,
        version=spec_version,
        openapi_version=openapi_version,
        plugins=(MarshmallowPlugin(),),
        **settings
    )

    validate_spec(spec)


Documenting Security Schemes
----------------------------

Use `spec.components.security_scheme <apispec.core.Components.security_scheme>`
to document `Security Scheme Objects <https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md#securitySchemeObject>`_.

.. code-block:: python

    from pprint import pprint
    from apispec import APISpec

    spec = APISpec(title="Swagger Petstore", version="1.0.0", openapi_version="3.0.2")

    api_key_scheme = {"type": "apiKey", "in": "header", "name": "X-API-Key"}
    jwt_scheme = {"type": "http", "scheme": "bearer", "bearerFormat": "JWT"}

    spec.components.security_scheme("api_key", api_key_scheme)
    spec.components.security_scheme("jwt", jwt_scheme)

    pprint(spec.to_dict()["components"]["securitySchemes"], indent=2)
    # { 'api_key': {'in': 'header', 'name': 'X-API-Key', 'type': 'apiKey'},
    #   'jwt': {'bearerFormat': 'JWT', 'scheme': 'bearer', 'type': 'http'}}

Referencing Top-level Components
--------------------------------

On OpenAPI, top-level component are meant to be referenced using a ``$ref``,
as in ``{$ref: '#/components/schemas/Pet'}`` (OpenAPI v3) or
``{$ref: '#/definitions/Pet'}`` (OpenAPI v2).

APISpec automatically resolves references in paths and in components themselves
when a string is provided while a dict is expected. Passing a fully-resolved
reference is not supported. In other words, rather than passing
``{"schema": {$ref: '#/components/schemas/Pet'}}``, the user must pass
``{"schema": "Pet"}``. APISpec assumes a schema reference named ``"Pet"`` has
been defined and builds the reference using the components location
corresponding to the OpenAPI version.
