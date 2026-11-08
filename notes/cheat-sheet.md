# Swagger/OpenAPI Cheat Sheet

## Concepts

An **HTTP API** is any application programming interface that uses HTTP as its underlying communication protocol to exchange data over a network or the internet.

Many **non-HTTP APIs** use other protocols to run over a network (MQTT, raw TCP sockets) or run locally on a machine or within an operating system's hardware layer. APIs such as Win32 (Windows), or Cocoa (macOS) let apps interact with the OS to open windows, print a page, save a Word document to a hard drive. Another application is graphics APIs, such as OpenGL, which GPU drivers implement, and programming language APIs (Java, Python, C++) to perform complex tasks locally without a network.

### Methods

Methods (*verbs*) perform an action on a resource. The convention is to write them in capitals.

- **GET** retrieves information.
- **HEAD** retrieves the metadata without the data itself (no response payload body).
- **PUT** creates or updates information.
- **POST** processes information (subscribe, complete a transaction).
- **DELETE** deletes information.
- **CONNECT** requests a TCP/IP tunnel to the origin server.
- **PATCH** applies partial modifications to a resource (save bandwidth over a PUT).
- **OPTIONS** requests which methods are supported.
- **QUERY** "requests that the request target process the enclosed content in a safe and idempotent manner and then respond with the result of that processing. This is similar to POST requests, but QUERY requests can be automatically repeated or restarted without concern for partial state changes." (QUERY is a Proposed Standard, [RFC 10008](https://www.rfc-editor.org/info/rfc10008/). OpenAPI 3.2 is the first version with a `query` operation.)
- **TRACE** "method requests a remote, application-level loop-back of the request message." [RFC 9110 9.3.8](https://www.rfc-editor.org/info/rfc9110/#section-9.3.8)

### Swagger and OpenAPI

**Swagger** was originally an open source specification for describing HTTP APIs. The Swagger project was acquired by SmartBear Software who donated the Swagger specification to the Linux Foundation when SmartBear and other vendors formed the OpenAPI Initiative (OAI). The OAI renamed the specification OpenAPI Specification (OAS) and maintains it as a vendor-neutral standard. The Swagger tools are open source while SmartBear markets SwaggerHub as a commercial product. The Swagger 2.0 spec is also called OpenAPI 2.0.

### Anatomy of an OpenAPI 3.0 document

Source: [OpenAPI Object](https://spec.openapis.org/oas/v3.0.4.html)

| Field | Required | Description |
|---|---|---|
|`openapi`| Yes | Specification version, such as `3.0.4` |
|`info`| Yes| Metadata about the API, `title` and `version` are required. |
|`servers`| No| Array of Server Objects, which provide connection info to a target server. If empty, default value is a Server Object with a url value of `/`  |
| `paths`| Yes | Available paths and operations for the API (`get`,`head`,`post`,...) |
|`components`| No| An element to hold various Objects for the OpenAPI Description: schemas, responses, parameters, etc.|
|`security`|No| A declaration of which security mechanisms can be used across the API. |
|`tags`|No| A list of tags used by the OpenAPI Description with additional metadata. This is used by Swagger UI for sections.|
|`externalDocs`| No| Links to additional external documentation |

## Swagger 2.0 vs OpenAPI 3.0

|Feature|Swagger 2.0|OpenAPI 3.0|
|---|---|---|
|Server Configuration|Separate root fields: `host`, `basePath`, and `schemes`|Flexible server array that supports multiple definitions and URL path templating with variables|
|Request Body and Parameters|Generic `in: body` for payload data and separate `formData` for form submissions|A `requestBody` object that also handles form data using standard multipart or URL-encoded media types|
|Reusable components|Separate fields: `definitions`,`parameters`,`responses`, and `securityDefinitions`|Reusable components centralized in a `components` object|
|Content Negotiation|Uses global or operation-level `consumes` and `produces` arrays |Moves content handling into a local `content` map in each request body and response, with unique schemas per media type|
|Security & Authentication|Authentication: `basic`, `apiKey`, and `oauth2` | Adds `http` (bearer; basic moves here), `openIdConnect`, and `apiKey` `in: cookie` |

These differences between the two specifications are important to understand because enterprise platforms often serve both `/swagger.json` and `/openapi.json` to maximize compatibility with their clients.

### 1. Server Configuration

#### Swagger 2.0

Swagger 2.0 can only support one host.

```yaml
swagger: "2.0"

host: api.example.com
basePath: /v2.0
schemes:
  - https
```

#### OpenAPI 3.0

OpenAPI 3.0 supports the use of variables in server configurations.

```yaml
openapi: "3.0.3"

servers:
  - url: https://{subdomain}.example.com/{version} # Variables are new in OpenAPI 3.0
    description: Production server with variable subdomain and versions
    variables:
      subdomain:
        default: api
        description: The subdomain for the environment
      version:
        enum:
          - v1.8
          - v2.0
          - v2.1
        default: v2.0

```

Full examples: [2.0](examples/01-servers.v2.yaml) · [3.0](examples/01-servers.v3.yaml)

### 2. Request Body and Parameters

Swagger 2.0 requires two operations to do what OpenAPI 3.0 can do in one.

#### Swagger 2.0

Swagger 2.0 cannot have both body and formData appear in the same operation and so these are presented as distinct operations. Swagger 2.0 does not support two `POST /albums` operations, so these would need different paths (shown below), or separate specification files.

```yaml
swagger: "2.0"

paths:
  /albums:
    post:
      summary: Add a new album
      operationId: createAlbum
      consumes:
        - application/json
        - application/xml
      parameters:
        - in: body
          name: album
          description: New album
          required: true
          schema:
            $ref: '#/definitions/Album'
  /albums/form:
    post:
      summary: Submit a form to add a new album
      operationId: createAlbumFromForm
      consumes:
        - application/x-www-form-urlencoded
        - multipart/form-data
      parameters:                                  # each field is its own parameter, no schema
        - name: upc
          in: formData
          type: string
          pattern: '^\d{12,13}$'
          description: Universal Product Code or EAN barcode for the release
          required: true
        - name: albumName
          in: formData
          type: string
          description: The name of the album
          required: true
          # ...
        - name: submitDate
          in: formData
          type: string
          format: date-time
```

#### OpenAPI 3.0

```yaml
openapi: "3.0.3"

components:
  schemas:
    AlbumForm:                                  # the form's schema can sit in the components or inline under the media type
      type: object
      properties:
        upc:
          type: string
          pattern: '^\d{12,13}$'
          description: Universal Product Code or EAN barcode for the release
        albumName:
          type: string
        # ...
        submitDate:
          type: string
          format: date-time
      required:
        - upc
        - albumName
paths:
  /albums:
    post:
      summary: Add a new album
      operationId: createAlbum
      requestBody:
        description: Submit information to add a new album
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Album"
          application/xml:
            schema:
              $ref: "#/components/schemas/Album"
          application/x-www-form-urlencoded:
            schema:
              $ref: "#/components/schemas/AlbumForm"
          text/plain:                            # different schema per media type, in one operation
            schema:
              type: string

```

Full examples: [2.0](examples/02-request-body.v2.yaml) · [3.0](examples/02-request-body.v3.yaml)

### 3. Reusable components

#### Swagger 2.0

```yaml
swagger: "2.0"

definitions:
  User:
    type: object
    # ...
parameters:
  UserIdParam:
    name: userId
    in: path
    required: true
    type: integer                                    # type sits in the parameter
responses:
  Standard404Error:
    description: Resource not found
    schema:
      $ref: '#/definitions/ErrorModel'
securityDefinitions:
  ApiKeyAuth:
    type: apiKey
    # ...
```

#### OpenAPI 3.0

```yaml
openapi: "3.0.3"

components:                                          # one parent for all four
  schemas:
    User:
      type: object
      # ...
  parameters:
    UserIdParam:
      name: userId
      in: path
      required: true
      schema:
        type: integer                               # type moves under schema
        # ...
  responses:
    Standard404Error:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorModel'
  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      # ...
```

Full examples: [2.0](examples/03-components.v2.yaml) · [3.0](examples/03-components.v3.yaml)

### 4. Content Negotiation

In Swagger 2.0, every listed media type shares one schema. In OpenAPI 3.0, each media type can have its own unique schema.

#### Swagger 2.0

```yaml
swagger: "2.0"

consumes:
  - application/json
  - application/xml
produces:
  - application/json
  - application/xml

paths:
  /user:
    put:
      summary: Create or update a user
      description: Create a new user or update an existing user
      parameters:
        - name: user
          in: body
          schema:
            $ref: '#/definitions/User'
      responses:
        '200':
          description: A single user
          schema:
            $ref: '#/definitions/User'

definitions:
  User:                                             # a single schema used for all media types
    type: object
    # ...

```

#### OpenAPI 3.0

```yaml
openapi: "3.0.3"

components:
  schemas:
    User:
      type: object
      # ...
    UserBasic:                                     # can have separate schema for media types
      type: object
      # ...

paths:
  /user:
    put:
      summary: Create or update a user
      description: Create a new user or update an existing user
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/User'
          application/xml:
            schema:                                  # media type-specific schema
              $ref: '#/components/schemas/UserBasic'
      responses:
        '200':
          description: A single user
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
            application/xml:
              schema:                                # media type-specific schema
                $ref: '#/components/schemas/UserBasic'

```

Full examples: [2.0](examples/04-content.v2.yaml) · [3.0](examples/04-content.v3.yaml)

### 5. Security & Authentication

#### Swagger 2.0

```yaml
swagger: "2.0"

security:
  - oauth2Auth: [read]
  - httpBasicAuth: []
    apiKeyAuth: []

securityDefinitions:
  oauth2Auth:
    type: oauth2
    flow: accessCode
    authorizationUrl: https://example.com/oauth2/dialog
    tokenUrl: https://example.com/oauth/token
    scopes:
      read: Read user profiles.
  httpBasicAuth:
    type: basic
  # ...
```

#### OpenAPI 3.0

```yaml
openapi: "3.0.3"

security:
  - oauth2Auth: [read]
  - httpBasicAuth: []
    apiKeyAuth: []
  - httpBearerAuth: []                             # new authentication method
  - openIdConnect: []                              # new authentication method
    cookieAuth: []
components:
  securitySchemes:
    oauth2Auth:
      type: oauth2
      flows:
        authorizationCode:                          # changed from accessCode
          authorizationUrl: https://example.com/oauth2/dialog
          tokenUrl: https://example.com/oauth/token
          scopes:
            read: Read user profiles.
    httpBasicAuth:
      type: http                                   # changed from type: basic to type: http and scheme: basic
      scheme: basic
    # ...
    httpBearerAuth:                                 # new authentication method
      type: http
      scheme: bearer
      bearerFormat: JWT
    openIdConnect:                                  # new authentication method
      type: openIdConnect
      openIdConnectUrl: https://example.com/openidconnect
    cookieAuth:
      type: apiKey
      in: cookie                                     # new in value
      name: EXAMPLE-SESSION-ID
```

Full examples: [2.0](examples/05-security.v2.yaml) · [3.0](examples/05-security.v3.yaml)

## OpenAPI 3.1 Changes

OpenAPI 3.1 focused on modernizing the data model and schema capabilities. Major changes included:

- **Superset of JSON Schema draft 2020-12**. This allows JSON Schema validators and OpenAPI tools to share data models without translation, addresses "null" (below), and allows the use of JSON Schema features (`const`, `if/then/else`, `$defs`, etc.) inside API definitions.
- **Null became a type**. `nullable` is removed and type can now be a string `type: "null"` or array, `type: [string, "null"]`.
- **`webhooks` field added**. A new top-level field for describing asynchronous events that consumers receive from an API.
- **Dialects became explicit**. A document can set `jsonSchemaDialect` to specify default schema dialects, and individual schemas can declare `$schema` so one document can mix schemas written against different drafts.

## OpenAPI 3.2 Changes

OpenAPI 3.2 extends what a document can describe, including hierarchical tags, a parameter for the whole query string, streaming and sequential media types, a `query` operation, and OAuth2 `deviceAuthorization` flow. 3.2 is designed as an upgrade from 3.1, although the spec allows minor versions to include low-impact breaking changes.

## Debugging

- Use the Chrome DevTools > Console to fetch an API, such as a sanity check like this one that returned a count of the vehicle makes from the NHTSA vPIC API

    ```js
    fetch("https://vpic.nhtsa.dot.gov/api/vehicles/GetMakesForVehicleType/car?format=json").then(r => r.json()).then(d => console.log(d.Count, d.Message))
    ```

  which responded on 11/06/26 with:

    ```js
    195 'Response returned successfully'
    ```
