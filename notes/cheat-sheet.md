# Swagger/OpenAPI Cheat Sheet

## Concepts

An **HTTP API** is any application programming interface that uses HTTP as its underlying communication protocol to exchange data over a network or the internet. 

**Non-HTTP APIs** run locally on a machine or within an operating system's hardware layer. APIs such as the Win32 (Windows), or Cocoa (Macs) let apps interact with the OS to open windows, print a page, save a Word document to a hard drive. Another application is hardware drivers such as graphics cards (OpenGL) for video games, and programming language APIs (Java, Python, C++) to perform complex tasks locally without a network.

### Methods

Methods (*verbs*) perform an action on a resource. They are case insensitive.
- **GET** methods retrieve information.
- **HEAD** methods retrieves the metadata without the data itself (no response payload body).
- **PUT** methods create or update information.
- **POST** methods process information (subscribe, complete a transaction).
- **DELETE** methods delete information.
- **CONNECT** method requests a TCP/IP tunnel to the origin server.
- **PATCH** method asks to modify a resource according to its partial state (save bandwidth over a PUT).
- **OPTIONS** method requests which methods are supported.
- **QUERY** method requests the server process the content in a safe and idempotentn way and respond with the result. Like a POST, but can be repeated or restarted without being concerned about partial state changes.

### Swagger and OpenAPI

**Swagger** was originally an open source specification for describing HTTP APIs. The Swagger project was acquired by SmartBear Software who donated the Swagger specification to the Linux Foundation and renamed OpenAPI. **OpenAPI 3.0** and **Swagger 2.0** are the same specification. The OpenAPI Initiative maintains the OpenAPI spec as a vendor-neutral standard. While SmartBear markets the Swagger Editor, Swagger UI, and Swagger Codegen tools as a commercial product.

### Anatomy of an OpenAPI 3.0 document

Source: [Swagger 3.0 OpenAPI Object](https://swagger.io/specification/v3/#openapi-object)
| Field | Required | Type |
|---|---|---|
|`openapi`| Yes | Specification version, such as `3.0.4`, `3.1.1` |
|`info`| Yes| Metadata about the API, `title` and `version` are required. |
|`servers`| No| Array of Server Objects, which provide connection info to a target server. If empty, default value is a Server Object with a url value of `/`.  |
| `paths`| Yes | Available paths and operations for the API (`get`,`head`,`post`,...) |
|`components`| No| An element to hold various Objects for the OpenAPI Description: schemas, responses, parameters, etc.|
|`security`|No| A declaration of which security mechanisms can be used across the API (`openIdConnect`,`oauth2`). |
|`tags`|No| A list of tags used by the OpenAPI Description with additional metadata. This is used by Swagger UI for sections.|
|`externalDocs`| No| Links to additional external documentation |

### Swagger 2.0 vs OpenAPI 3.0

|Feature|Swagger 2.0|OpenAPI 3.0|
|---|---|---|
|Server Configuration|Separate root fields:  `host`, `basePath`, and `schemes`|Flexible server array that supports multiple definitions and URL path templating with variables|
|Request Body and Parameters|Generic `in:body` for payload data and separate `formData` for form submissions|`requestBody` object and handles form data using standard multipart or URL-encoded media types|
|Reusable components|Separate fields: `definitions`,`parameters`,`responses`, and `securityDefinitions`|Reuseable components centralized in a `components` object|
|Content Negotiation|Uses global or operation-level `consumes` and `produces` arrays |Moves content handling into a local `content` map in each request body and response, with unique schemas per media type|
|Security & Authentication|Authentication: `basic`, `apiKey`, and `oauth2` |`apiKey`, `http` (covers basic and bearer), `oauth2`, and `openIdConnect`. Also adds `cookie`|

#### 1. Server Configuration

##### Swagger 2.0

Swagger 2.0 can only support one host.

```yaml
swagger: "2.0"
info:
  title: Swagger 2.0 Server Configuration 
  description: |
    This is an example of server configuration for Swagger 2.0 
  version: "1.0.0"

host: api.example.com
basePath: /v2.0
schemes:
   - https

paths: {}
```

##### OpenAPI 3.0

OpenAPI 3.0 supports the use of variables in server configurations.

```yaml
openapi: "3.0.3"
info:
  title: OpenAPI 3.0 Server Configuration 
  description: |
    This is an example of server configuration for OpenAPI 3.0
  version: "1.0.0"
  
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

paths: {}
```

#### 2. Request Body and Parameters

Swagger 2.0 requires separate operations to perform the same one that OpenAPI 3.0 can do in a single operation.

##### Swagger 2.0

Swagger 2.0 cannot have both body and formData appear in the same operation and so these are presented as distinct operations. Swagger 2.0 does not support two POST/albums operations, so these would need different paths, or separate specification files.

**body**
```yaml
swagger: "2.0"
info:
  title: Swagger 2.0 Request Body and Parameters  
  description: |
    This is an example of request body and parameters configuration for Swagger 2.0 
  version: "1.0.0"

schemes:
  - https

consumes:
  - application/json
  - application/xml
produces:
  - application/json

paths:
  /albums:
    post:
      summary: Add a new album
      description: Adds a new album, with name, artist, and release year.
      parameters:
        - in: body
          name: message
          description: New album 
          required: true
          schema:
            $ref: '#/definitions/Album'
      responses:
        '200':
          description: Album created successfully
        '400':
          description: Invalid input
        '404':
          description: Resource not found

definitions:
  Album:
    type: object
    required:
      - upc
      - albumName
    properties:
        upc: 
          type: string
          pattern: '^\d{12,13}$'
          description: 'Universal Product Code or EAN barcode for the release'
        albumName:
          type: string
        artistName:
          type: string
        releaseYear:
          type: integer

```

**formData**
```yaml
swagger: "2.0"
info:
  title: Swagger 2.0 formData Configuration
  description: |
    This is an example of form data configuration for Swagger 2.0 
  version: "1.0.0"

paths:
  /albums:
    post:
      summary: Submit form data for a album
      consumes:
        - application/x-www-form-urlencoded
        - multipart/form-data
      parameters:
        - name: upc
          in: formData
          type: string
          pattern: '^\d{12,13}$'
          description: 'Universal Product Code or EAN barcode for the release'
          required: true
        - name: albumName
          in: formData
          type: string
          description: The name of the album
          required: true
        - name: artistName
          in: formData
          type: string
          description: The name of the artist or band
        - name: releaseYear
          in: formData
          type: integer
          description: The year the album was officially released
        - name: submitDate
          in: formData
          type: string
          format: date-time
      responses:
        '200':
          description: Submitted successfully
        '400':
          description: Invalid input
        '404':
          description: Resource not found
```

##### OpenAPI 3.0

```yaml
openapi: "3.0.3"
info:
  title: OpenAPI 3.0 requestBody and Parameters
  description: |
    This is an example of how to configure requestBody and parameters in OpenAPI 3.0
  version: "1.0.0"

components:
  schemas:
    Album:
      type: object
      required:
        - upc
        - albumName
      properties:
        upc: 
          type: string
          pattern: '^\d{12,13}$'
          description: 'Universal Product Code or EAN barcode for the release'
        albumName:
          type: string
        artistName:
          type: string
        releaseYear:
          type: integer
    AlbumForm:
      type: object
      properties:
        upc: 
          type: string
          pattern: '^\d{12,13}$'
          description: 'Universal Product Code or EAN barcode for the release'
        albumName:
          type: string
        artistName:
          type: string
        releaseYear:
          type: integer
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
      operationId: postAlbumById
      requestBody:
        description: Submit information about a album
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
          text/plain: # <-- New to OpenAPI 3.0
            schema:
              type: string

      responses:
        '200':
          description: Successfully submitted
        '400':
          description: Invalid input
        '404':
          description: Resource not found
```

#### 3. Reusable components

##### Swagger 2.0
```yaml
swagger: '2.0'
info:
  title: Swagger 2.0 Resuable Components Configuration
  description: |
    This is an example of reusable components configuration for Swagger 2.0 
  version: "1.0.0"

definitions:
  ErrorModel:
    type: object
    required:
      - code
      - message
    properties:
      code:
        type: integer
        format: int32
      message:
        type: string

  User:
    type: object
    properties:
      id:
        type: integer
        format: int64
      username:
        type: string
      email:
        type: string

parameters:
  UserIdParam:
    name: userId
    in: path
    description: ID of the user to fetch
    required: true
    type: integer
    format: int64
  LimitParam:
    name: limit
    in: query
    description: Max number of items to return
    required: false
    type: integer
    default: 20

responses:
  Standard400Error:
    description: Invalid input / Bad request
    schema:
      $ref: '#/definitions/ErrorModel'

  Standard404Error:
    description: Resource not found
    schema:
      $ref: '#/definitions/ErrorModel'

securityDefinitions:
  ApiKeyAuth:
    type: apiKey
    name: X-API-KEY
    in: header

paths:
  /users/{userId}:
    get:
      summary: Get user by ID
      parameters:
        - $ref: '#/parameters/UserIdParam'
        - $ref: '#/parameters/LimitParam'
      security:
        - ApiKeyAuth: []
      responses:
        200:
          description: A single user
          schema:
            $ref: '#/definitions/User'
        400:
          $ref: '#/responses/Standard400Error'
        404:
          $ref: '#/responses/Standard404Error'
```

##### OpenAPI 3.0
```yaml
openapi: "3.0.3"
info:
  title: OpenAPI 3.0 Resuable Components Configuration
  description: |
    This is an example of reusable components configuration for OpenAPI 3.0
  version: "1.0.0"

components:
  schemas:
    ErrorModel:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string
    User:
      type: object
      properties:
        id:
          type: integer
          format: int64
        username:
          type: string
        email:
          type: string

  parameters:
    UserIdParam:
      name: userId
      in: path
      description: ID of the user to fetch
      required: true
      schema: 
        type: integer
        format: int64
    LimitParam:
      name: limit
      in: query
      description: Max number of items to return
      required: false
      schema: 
        type: integer
        default: 20

  responses:
    Standard400Error:
      description: Invalid input / Bad request
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorModel'

    Standard404Error:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorModel'

  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      name: X-API-KEY
      in: header

paths:
  /users/{userId}:
    get:
      description: Returns user based on ID
      summary: Get user by ID
      operationId: getUsersById
      parameters:
        - $ref: '#/components/parameters/UserIdParam'
        - $ref: '#/components/parameters/LimitParam'        
      security:
        - ApiKeyAuth: []
      responses:
        '200':
          description: A single user
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/Standard400Error'
        '404':
          $ref: '#/components/responses/Standard404Error'
```

#### 4. Content Negotiation

In Swagger 2.0, every listed media type shares one schema. In OpenAPI 3.0, each media type can have its own unique schema.

##### Swagger 2.0

```yaml
swagger: "2.0"
info:
  title: Swagger 2.0 Content Negotiation Configuration
  description: |
    This is an example of content negotiation configuration for Swagger 2.0 
  version: "1.0.0"

consumes:
  - application/json
  - application/xml
produces:
  - application/json
  - application/xml

paths: 
  /user:
    put:
      summary: Create or update users
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
        '400':
          $ref: '#/responses/Standard400Error'
        '404':
          $ref: '#/responses/Standard404Error'

responses:
  Standard400Error:
    description: Invalid input / Bad request
    schema:
      $ref: '#/definitions/ErrorModel'

  Standard404Error:
    description: Resource not found
    schema:
      $ref: '#/definitions/ErrorModel'

definitions:
  ErrorModel:
    type: object
    required:
      - code
      - message
    properties:
      code:
        type: integer
        format: int32
      message:
        type: string

  User:
    type: object
    properties:
      id:
        type: integer
        format: int64
      username:
        type: string
      email:
        type: string

```

##### OpenAPI 3.0

```yaml
openapi: "3.0.3"
info:
  title: Content Negotiation Example
  description: |
    This is an example of content negotiation for OpenAPI 3.0
  version: "1.0.0"

components:
  schemas:
    ErrorModel:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string
    User:
      type: object
      properties:
        id:
          type: integer
          format: int64
        username:
          type: string
        email:
          type: string
    UserBasic:  # <-- New to OpenAPI 3.0
      type: object
      properties:
        id:
          type: integer
          format: int64
        username:
          type: string

  responses:
    Standard400Error:
      description: Bad request or invalid input
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorModel'
        application/xml:
          schema:
            $ref: '#/components/schemas/ErrorModel'
    Standard404Error:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorModel'
        application/xml:
          schema:
            $ref: '#/components/schemas/ErrorModel'
paths:
  /user:
    put:
      tags:
        - Users
      summary: Creates or updates a user
      description: Creates a new user or updates an existing user
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/User'
          application/xml:
            schema:
              $ref: '#/components/schemas/UserBasic'
      responses:
        '200':
          description: A single user
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
            application/xml:
              schema:
                $ref: '#/components/schemas/UserBasic'
        '400':
          $ref: '#/components/responses/Standard400Error'
        '404':
          $ref: '#/components/responses/Standard404Error'
      
```

#### 5. Security & Authentication

##### Swagger 2.0

```yaml
swagger: "2.0"
info:
  title: Security and Authentication Example
  description: |
    Simple example of API security for Swagger 2.0
  version: "1.0.0"

host:  api.example.com
schemes: 
  - https
basePath: /v2.0

consumes:
  - application/json
produces:
  - application/json
security:
  - oauth2Auth: [uid] 
  - httpBasicAuth: []
    apiAuth: [] 

paths: 
  /items:
    get:
      summary: List users
      responses:
        200:
          description: Success

securityDefinitions:
  oauth2Auth:
    type: oauth2
    flow: accessCode
    authorizationUrl: https://example.com/oauth2/dialog
    tokenUrl: https://example.com/oauth/token
    scopes:
      uid: Unique identifier of the user accessing the service.
  httpBasicAuth:
    type: basic
  apiAuth:
    type: apiKey
    in: header
    name: EXAMPLE-API-KEY
```

##### OpenAPI 3.0

```yaml
openapi: "3.0.3"
info:  
  title: Security and Authentication Example
  description: |
    This is an **example** API to demonstrate security and authentication features of OpenAPI 3.0.
  version: "1.0.0"

servers:
  - url: https://{subdomain}.example.com/{version} 
    description: Production server with variable subdomain and versions
    variables:
      subdomain:
        default: api
        description: The subdomain for the environment
      version:
        enum:
          - v1.8
          - v2.0
        default: v2.0

security:
  - apiKeyAuth: []
    httpBasicAuth: []
  - oauth2Auth: [uid]
  - httpBearerAuth: [] # <-- New to OpenAPI 3.0
  - openIdConnect: [] # <-- New to OpenAPI 3.0
  - cookieAuth: []  # <-- New to OpenAPI 3.0

paths: 
  /items:
    get:
      summary: List users
      operationId: getUsers
      responses:
        200:
          description: Success

components:
  securitySchemes:
    apiKeyAuth: 
      type: apiKey
      in: header
      name: EXAMPLE-API-KEY
    oauth2Auth:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://example.com/oauth2/dialog  
          tokenUrl: https://example.com/oauth/token
          scopes:
            uid: Unique identifier of the user accessing the service.
    httpBearerAuth: # <-- New to OpenAPI 3.0 
      type: http
      scheme: bearer
      bearerFormat: JWT
    httpBasicAuth: 
      type: http
      scheme: basic
    openIdConnect: # <-- New to OpenAPI 3.0
      type: openIdConnect
      openIdConnectUrl: https://example.com/openidconnect
    cookieAuth:  # <-- New to OpenAPI 3.0
      type: apiKey
      in: cookie
      name: EXAMPLE-SESSION-ID
```

## Debugging 
- Use the Chrome Dev Tools > Console to fetch an API, such as a sanity check like this one that returned a count of the vehicle makes from the NHTSA vpic API
    ```
    fetch("https://vpic.nhtsa.dot.gov/api/vehicles/GetMakesForVehicleType/car?format=json").then(r => r.json()).then(d => console.log(d.Count, d.Message))
    ```
    which responded with:  `195 'Response returned successfully'` 
