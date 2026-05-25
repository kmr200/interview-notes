# REST Basics

## What is REST?

REST (Representational State Transfer) is an architectural style that defines a set of constraints for building web services. All communication in a REST API uses HTTP exclusively.

REST is preferred over SOAP in many contexts because it uses less bandwidth and is simpler and more flexible, making it well-suited for internet-scale usage.

---

## Richardson Maturity Model

The Richardson Maturity Model grades REST APIs by how closely they follow REST constraints, from Level 0 (least mature) to Level 3 (fully RESTful).

| Level | Name                | Description                                                                                                                       |
|-------|---------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| **0** | The Swamp of POX    | Single URI, single HTTP method (typically POST). No use of URIs, HTTP verbs, or HATEOAS.                                          |
| **1** | Resources           | Different URIs per resource, but still a single HTTP verb (typically POST). Each resource has a unique URI.                       |
| **2** | HTTP Verbs          | Different URIs and different HTTP methods (GET, POST, PUT, DELETE). No HATEOAS. This is the most common level in practice.        |
| **3** | Hypermedia Controls | Uses HATEOAS - responses include links that guide clients to related resources, making the API self-descriptive and discoverable. |

---

## Key Definitions

### Resource
The key abstraction in REST - any named concept: a document, image, service, collection, or object. A resource is identified by a URI and has a type, data, relationships, and a small set of standard methods (GET, POST, PUT, DELETE).

- **Singleton resource**: a single item - `/customers/{customerId}`
- **Collection resource**: a directory of items - `/customers`
- **Subcollection**: resources nested under another - `/customers/{customerId}/accounts`

### Resource State
The current state of a resource on the server at any point in time. What the server returns in a response is called a **resource representation**.

### Resource Representations
The same resource can be represented in multiple formats (JSON, XML, YAML, etc.). Clients use content negotiation (`Accept` / `Content-Type` headers) to request a specific format.

### Application State
Server-side data that identifies a client session and its context. **Not** the same as resource state - they are completely separate concepts.

### Statelessness
Every HTTP request must contain all information needed for the server to fulfill it. The server stores no session data about the client between requests. This improves availability and scalability but increases request payload size.

### Idempotence
A method is **idempotent** if calling it multiple times produces the same result as calling it once.

```
a = 5;   // idempotent - result is always a = 5
a++;     // NOT idempotent - result depends on how many times it runs
```

> POST is **neither** safe nor idempotent.

### Safety
A method is **safe** if it does not modify the resource - it only reads.

```
x + 0;   // idempotent AND safe - no change, same result every time
x = 5;   // idempotent but NOT safe - changes x if x ≠ 5
```

> All safe methods are idempotent, but not all idempotent methods are safe.

### PUT vs PATCH

|            | PUT                            | PATCH                |
|------------|--------------------------------|----------------------|
| Scope      | Full replacement of a resource | Partial modification |
| Idempotent | Always                         | Not necessarily      |
| Safe       | No                             | No                   |

PATCH uses a patch language (e.g., JSON Patch) to describe changes, not just a modified subset of fields. Like POST, PATCH may have side effects on other resources.

### HATEOAS (Hypermedia as the Engine of Application State)
A REST constraint where API responses include hypermedia links to related resources, allowing clients to navigate the API dynamically without hardcoded URIs.

Example response from `GET /management/departments/10`:
```json
{
  "departmentId": 10,
  "departmentName": "Administration",
  "locationId": 1700,
  "managerId": 200,
  "links": [
    {
      "href": "10/employees",
      "rel": "employees",
      "action": "GET",
      "types": ["text/xml", "application/json"]
    }
  ]
}
```

The client follows the `href` link to retrieve employees - the server drives navigation, not the client.

---

## REST Architectural Constraints

### 1. Client-Server
Client and server must be able to evolve independently. The client knows only resource URIs; the server knows nothing about the UI. Neither depends on the other's implementation.

### 2. Uniform Interface
The key constraint that distinguishes REST from non-REST. Four principles:
- **Resource-based** - resources are identified in requests via URIs.
- **Manipulation through representations** - the client holds enough information to modify or delete a resource.
- **Self-descriptive messages** - each message contains enough information to describe how to process it.
- **HATEOAS** - responses include links for the client to discover related resources.

### 3. Stateless
The server stores no client session data. Every request is self-contained. Authentication credentials must be sent with every request. Improves availability and scalability; a drawback is increased bandwidth usage.

### 4. Cacheable
Responses must declare whether they are cacheable and for how long. Proper caching reduces client-server interactions and improves performance. Stale data risk must be managed by invalidating the cache when server data changes.

### 5. Layered System
The architecture can have multiple layers (load balancers, caches, gateways) between client and server, and neither party needs to know about intermediate layers. This improves scalability and availability.

### 6. Code on Demand *(optional)*
Servers may send executable code to clients (e.g., JavaScript widgets, client-side scripts). This is the only optional constraint.

---

## REST API Design Principles

### 1. Use nouns for resources, not verbs

URIs identify resources (things), not actions. HTTP methods express the action.

```
Good:
GET  /device-management/managed-devices
POST /device-management/managed-devices

Bad:
GET  /device-management/getDevices
```

**Resource archetypes:**

| Archetype                             | Naming        | Example URI                                              |
|---------------------------------------|---------------|----------------------------------------------------------|
| **Document** (singular concept)       | Singular noun | `/managed-devices/{id}`, `/users/admin`                  |
| **Collection** (server-managed list)  | Plural noun   | `/managed-devices`, `/users`                             |
| **Store** (client-managed repository) | Plural noun   | `/users/{id}/playlists`                                  |
| **Controller** (procedural action)    | Verb          | `/users/{id}/cart/checkout`, `/users/{id}/playlist/play` |

### 2. Consistent URI formatting

| Rule                           | Good                    | Bad                                       |
|--------------------------------|-------------------------|-------------------------------------------|
| Use `/` for hierarchy          | `/devices/{id}/scripts` | -                                         |
| No trailing slash              | `/managed-devices`      | `/managed-devices/`                       |
| Use hyphens for readability    | `/device-management`    | `/deviceManagement`, `/device_management` |
| Lowercase only                 | `/my-folder/my-doc`     | `/My-Folder/my-doc`                       |
| No file extensions             | `/managed-devices`      | `/managed-devices.xml`                    |
| No CRUD verbs in URI           | `DELETE /devices/{id}`  | `GET /deleteDevice/{id}`                  |
| Use query params for filtering | `/devices?region=USA`   | `/devices/region/USA`                     |

### 3. HTTP methods and status codes

Use HTTP verbs to express the operation, not the URI:

```
GET    /devices        - list all
POST   /devices        - create new
GET    /devices/{id}   - get one
PUT    /devices/{id}   - full update
PATCH  /devices/{id}   - partial update
DELETE /devices/{id}   - delete
```

**Common HTTP status codes:**

| Code                        | Meaning                                                               |
|-----------------------------|-----------------------------------------------------------------------|
| `200 OK`                    | Request succeeded.                                                    |
| `201 Created`               | Resource successfully created.                                        |
| `204 No Content`            | Success with no response body (e.g. DELETE).                          |
| `400 Bad Request`           | Malformed request or invalid JSON.                                    |
| `401 Unauthorized`          | Not authenticated.                                                    |
| `403 Forbidden`             | Authenticated but not permitted.                                      |
| `404 Not Found`             | Resource does not exist.                                              |
| `406 Not Acceptable`        | Server cannot produce the requested content type.                     |
| `409 Conflict`              | Request conflicts with current resource state (e.g. duplicate).       |
| `422 Unprocessable Entity`  | Syntax is correct but semantics are invalid (e.g. validation errors). |
| `500 Internal Server Error` | Generic server-side error.                                            |
| `502 Bad Gateway`           | Invalid response from upstream server.                                |
| `503 Service Unavailable`   | Server overloaded or partially unavailable.                           |

### 4. Filtering, sorting, pagination, and searching

All implemented as query parameters on the base resource URI - don't create new endpoints.

**Filtering:**
```
GET /tickets?state=open
GET /devices?region=USA&brand=XYZ
```

**Sorting:**
```
GET /tickets?sort=-priority              // descending priority
GET /tickets?sort=-priority,createdAt   // descending priority, then oldest first
GET /users?sort=email&order=asc
```

**Pagination:**
```
GET /companies?limit=20                       // first 20
GET /companies?limit=20&offset=20            // next 20
GET /companies?limit=20&after_id=20          // cursor-based
GET /companies?page=23
```

**Searching:**
```
GET /tickets?q=return&state=open&sort=-priority,createdAt
GET /items?price[gte]=10&price[lte]=100
GET /items?price=gte:10&price=lte:100
```

**Field selection** (reduce response payload):
```
GET /tickets?fields=id,subject,updated_at&state=open
```

**Embedding related resources:**
```
GET /tickets/12?embed=customer.name,assigned_user
```

**Aliases for common queries** (when URIs get too long):
```
GET /tickets/recently_closed
GET /tickets?queryAlias=recently_closed
```

### 5. Error handling

Always return meaningful HTTP status codes and a structured error body.

**Standard error response:**
```json
{
  "code": 1234,
  "message": "Something went wrong",
  "description": "More details about the error"
}
```

**Validation error response** (for PUT, PATCH, POST):
```json
{
  "code": 1024,
  "message": "Validation Failed",
  "errors": [
    {
      "code": 5432,
      "field": "first_name",
      "message": "First name cannot have fancy characters"
    },
    {
      "code": 5622,
      "field": "password",
      "message": "Password cannot be blank"
    }
  ]
}
```

### 6. Security

- **Always use HTTPS** - enables simple token-based auth via HTTP Basic Auth.
- **Hash passwords** - use PBKDF2, bcrypt, or scrypt.
- **Never expose credentials in URIs** - API keys, tokens, and passwords in URLs are captured in logs.
- **Consider OAuth 2.0** - for delegated third-party access.
- **Add request timestamps** - reject requests older than a reasonable window (e.g., 30 seconds) to prevent replay attacks.
- **Validate all input** - reject invalid requests immediately, before reaching business logic.

### 7. Caching

- `GET` requests are cacheable by default.
- `POST` requests are not cacheable by default (can be made cacheable via `Expires` or `Cache-Control`).
- `PUT` and `DELETE` responses are never cacheable.

**Key caching headers:**

| Header          | Purpose                                                  | Example                                        |
|-----------------|----------------------------------------------------------|------------------------------------------------|
| `Expires`       | Absolute expiry time for cached response                 | `Expires: Fri, 20 May 2016 19:20:49 GMT`       |
| `Cache-Control` | Directives for caching behavior                          | `Cache-Control: max-age=3600`                  |
| `ETag`          | Opaque token identifying the current state of a resource | `ETag: "abcd1234567n34jv"`                     |
| `Last-Modified` | When the resource last changed                           | `Last-Modified: Fri, 10 May 2016 09:17:49 GMT` |

Cacheable responses should include either an `ETag` or `Last-Modified` header as a validator.

### 8. HATEOAS

The primary benefit of HATEOAS is **loose coupling**. If clients hardcode URIs, they are tightly coupled to the server's URL structure. When the server returns links in responses, clients discover URIs dynamically and are not affected by URI changes.

See the [HATEOAS definition above](#hateoas-hypermedia-as-the-engine-of-application-state) for an example.
