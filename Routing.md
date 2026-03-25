# 6. What is Routing in Backend? How Requests Find Their Way Home

## Table of Contents
1. [What is Routing?](#1-what-is-routing)
2. [Types of Routes](#2-types-of-routes)
3. [Path Parameters vs. Query Parameters](#3-path-parameters-vs-query-parameters)
4. [Nested Routing](#4-nested-routing)
5. [Route Versioning and Deprecation](#5-route-versioning-and-deprecation)
6. [Catch-All Routes](#6-catch-all-routes)
7. [Best Practices](#7-best-practices)
8. [Real-World Examples](#8-real-world-examples)

---

## 1. What is Routing?

### The Concept: "What" vs. "Where"

In a backend system, HTTP methods (like `GET`, `POST`, `DELETE`, `PUT`) express the **what**—the intent or action of a request (e.g., fetching, adding, updating, or deleting data). Routing expresses the **where**—the specific resource or destination you want to apply that action to.

### Definition

**Routing** is the process of mapping a combination of:
- An **HTTP method** (verb)
- A **URL path** (resource identifier)

...to a specific server-side **Handler** (a function or set of instructions that contains your business logic).

### Uniqueness Through Method + Path Combination

The server creates a **unique key** by concatenating the HTTP method and the route path. This allows:

```
GET /api/books       → Fetch all books (different handler)
POST /api/books      → Create a new book (different handler)
DELETE /api/books/5  → Delete book with ID 5 (different handler)
```

Even though all three requests target `/api/books`, the **HTTP method + path combination** ensures they trigger completely different logic without clashing.

### Why This Matters

Without routing, a backend server would be unable to:
- Distinguish between different types of requests to the same resource
- Direct requests to the correct business logic
- Maintain a semantic, predictable API structure

---

## 2. Types of Routes

Routes are structured in two primary ways:

### Static Routes

**Definition:** Constant URL paths that do not contain any variable parameters.

**Characteristics:**
- Remain identical regardless of client input
- Always point to the same resource or resource type
- Simple and predictable

**Examples:**
```
/api/books
/api/health-check
/api/config/settings
/api/auth/logout
```

### Dynamic Routes

**Definition:** URL paths that include variable slots (placeholders) that the server can extract as data.

**Syntax:**
- Most frameworks use a **colon prefix** for dynamic segments: `:parameter_name`
- Some frameworks use **curly braces**: `{id}` or square brackets `[id]`

**Examples:**
```
/api/users/:id
/api/posts/:postId
/api/users/:userId/comments/:commentId
/api/products/{productId}/reviews
```

**How It Works:**

When a client requests `/api/users/123`:
1. The router matches the pattern `/api/users/:id`
2. The server **extracts "123"** and assigns it to the variable `id`
3. The handler receives `id = 123` and uses it to fetch the specific user's data

**Advantages:**
- Flexible and scalable
- Reduces code duplication (one route handles infinite ID variations)
- Aligns with REST principles

---

## 3. Path Parameters vs. Query Parameters

Both are mechanisms for passing data to the server through the URL, but they serve different purposes:

### Path Parameters (Route Parameters)

**Definition:** Variables placed directly inside the route's path, after forward slashes `/`.

**Syntax:**
```
/api/users/123
/api/users/123/posts/456
/api/authors/john-doe/books
```

**Characteristics:**
- Extract specific resource identifiers
- Express **semantic hierarchy** and uniqueness
- Required for accessing that route

**Use Cases:**
- Identifying a **unique resource** (user ID, post ID, product ID)
- Building **nested resource hierarchies**
- Expressing **resource ownership** (user → their posts)

**Code Example (FastAPI/Python):**
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/api/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id, "name": "John"}

@app.get("/api/users/{user_id}/posts/{post_id}")
def get_user_post(user_id: int, post_id: int):
    return {"user_id": user_id, "post_id": post_id, "content": "..."}
```

### Query Parameters

**Definition:** Key-value pairs appended to the end of a URL after a question mark `?`.

**Syntax:**
```
/api/search?query=python
/api/posts?page=2&limit=20
/api/users?role=admin&status=active
```

**Characteristics:**
- **Optional** (not required to access the route)
- Multiple parameters separated by `&`
- Used for **filtering, sorting, and pagination**
- Ideal for `GET` requests (which don't have a request body)

**Use Cases:**
- **Pagination:** `/api/posts?page=2&limit=10`
- **Filtering:** `/api/products?category=electronics&price_min=100`
- **Sorting:** `/api/users?sort_by=name&order=asc`
- **Search:** `/api/search?q=backend+routing`
- **Metadata:** `/api/logs?date=2024-01-15&level=error`

**Code Example (FastAPI/Python):**
```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/api/posts")
def get_posts(page: int = Query(1), limit: int = Query(10), status: str = Query("published")):
    return {
        "page": page,
        "limit": limit,
        "status": status,
        "posts": [...]
    }

# Client request: /api/posts?page=2&limit=20&status=draft
```

### Path Parameters vs. Query Parameters: Quick Comparison

| Aspect | Path Parameters | Query Parameters |
|--------|-----------------|------------------|
| **Position** | Inside the path | After `?` |
| **Required** | Yes | No (optional) |
| **Purpose** | Identify specific resources | Filter, sort, paginate |
| **Example** | `/users/123` | `/users?role=admin` |
| **Semantics** | "Get user 123" | "Get users filtered by role" |

---

## 4. Nested Routing

Nested routing is a **standard REST API practice** used to express a hierarchical relationship between different resources.

### What Makes It Powerful

By nesting route paths, you create a highly readable, **semantic expression** of what data you want. The hierarchy in the URL mirrors the relationship in your data model.

### Example Workflow

Let's trace through a real-world scenario: fetching data about users and their posts.

```
GET /api/users
  └─ Fetch a list of ALL users

GET /api/users/123
  └─ Fetch the SPECIFIC user with ID 123

GET /api/users/123/posts
  └─ Fetch ALL posts created by user 123

GET /api/users/123/posts/456
  └─ Fetch the SPECIFIC post (ID 456) belonging to that specific user
```

**Why This Hierarchy Matters:**
- **Readability:** The URL tells you exactly what resource is being accessed
- **Logical Structure:** The nesting reflects the data relationship (users own posts)
- **RESTful Design:** Follows REST conventions for expressing resource ownership
- **Scalability:** Easy to extend further (e.g., `/api/users/123/posts/456/comments`)

### More Complex Example: E-Commerce

```
GET /api/stores/1/products
  └─ All products in store 1

GET /api/stores/1/products/99
  └─ Product 99 in store 1

GET /api/stores/1/products/99/reviews
  └─ All reviews for product 99 in store 1

GET /api/stores/1/products/99/reviews/5
  └─ Specific review 5 for product 99 in store 1

POST /api/stores/1/products/99/reviews
  └─ Create a new review for product 99 in store 1
```

### Code Example (FastAPI/Python)

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/api/users/{user_id}/posts")
def get_user_posts(user_id: int):
    # Fetch all posts for this specific user
    return {"user_id": user_id, "posts": [...]}

@app.get("/api/users/{user_id}/posts/{post_id}")
def get_user_post(user_id: int, post_id: int):
    # Fetch a specific post by a specific user
    # The nesting ensures the post belongs to this user
    return {"user_id": user_id, "post_id": post_id, "content": "..."}

@app.post("/api/users/{user_id}/posts")
def create_user_post(user_id: int, post_data: dict):
    # Create a new post for this specific user
    return {"user_id": user_id, "post_id": 789, "created": True}
```

### When NOT to Over-Nest

While nesting is powerful, don't nest more than **2-3 levels deep**. Deep nesting can become hard to maintain:

```
# ❌ Too deep - hard to maintain
GET /api/companies/1/departments/5/teams/3/members/7/projects/12

# ✅ Better - use IDs when the hierarchy is clear
GET /api/projects/12/members
GET /api/members/7
```

---

## 5. Route Versioning and Deprecation

As applications evolve, business requirements change. Sometimes you need to **completely alter** the format or structure of data your API returns—without breaking existing clients.

### The Problem: Breaking Changes

Imagine your original API returns:

```json
GET /api/products/1

{
  "name": "Laptop",
  "price": 999
}
```

Then your business decides to rename `name` → `title` and restructure the response:

```json
{
  "title": "Laptop",
  "pricing": {
    "amount": 999,
    "currency": "USD"
  }
}
```

**The Issue:** If you change the live route immediately:
- **iOS apps** expecting `name` will crash
- **React apps** expecting the old structure will break
- **Third-party integrations** will fail
- **Your users will experience broken functionality**

### The Solution: Route Versioning

Add **version numbers** to your routes:

```
GET /api/v1/products/1    → Old format (legacy)
GET /api/v2/products/1    → New format (current)
```

**Both routes coexist simultaneously:**

```python
from fastapi import FastAPI

app = FastAPI()

# Version 1 - Old format
@app.get("/api/v1/products/{product_id}")
def get_product_v1(product_id: int):
    return {
        "name": "Laptop",
        "price": 999
    }

# Version 2 - New format
@app.get("/api/v2/products/{product_id}")
def get_product_v2(product_id: int):
    return {
        "title": "Laptop",
        "pricing": {
            "amount": 999,
            "currency": "USD"
        }
    }
```

### Deprecation Strategy

1. **Release new version** (`v2`) while keeping old version (`v1`) live
2. **Announce deprecation** to all clients (via email, documentation, blog post)
3. **Provide migration timeline** (e.g., "v1 will be removed in 6 months")
4. **Monitor adoption** — track which clients are still using `v1`
5. **Eventually remove** the old version after the deadline

### Versioning Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL Path** | `/api/v1/users` | Clear, easy to test, discoverable | Duplication of code |
| **Query Parameter** | `/api/users?version=1` | Single endpoint, cleaner URLs | Less discoverable |
| **Header-Based** | `Accept: application/vnd.api+v1+json` | Clean URLs, semantic | Complex to implement |

**Recommendation:** Use **URL path versioning** for clarity and ease of testing.

---

## 6. Catch-All Routes

### Purpose and Definition

A **catch-all route** acts as a safety net for invalid or unmatched requests. It ensures that every request gets a response—even if it doesn't match any defined route.

### How It Works

1. Server receives a request
2. Router checks against all defined routes in order
3. If no route matches, the request **"trickles down"** through the routing logic
4. The catch-all route (usually placed last) **catches unmatched requests**
5. Returns a user-friendly error response (typically `404 Not Found`)

### Syntax and Implementation

Most frameworks use a **wildcard** syntax for catch-all routes:

```
/* or * or ** (depends on framework)
```

**Code Example (FastAPI/Python):**

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/api/users")
def get_users():
    return {"users": [...]}

@app.get("/api/products")
def get_products():
    return {"products": [...]}

# Catch-all route - placed at the END
@app.api_route("/{path:path}", methods=["GET", "POST", "PUT", "DELETE"])
def catch_all(path: str):
    return JSONResponse(
        status_code=404,
        content={
            "error": "Route Not Found",
            "message": f"The path '/{path}' does not exist",
            "status": 404
        }
    )
```

**Code Example (Express.js/Node.js):**

```javascript
const express = require('express');
const app = express();

app.get('/api/users', (req, res) => {
  res.json({ users: [...] });
});

app.get('/api/products', (req, res) => {
  res.json({ products: [...] });
});

// Catch-all route - placed at the END
app.use((req, res) => {
  res.status(404).json({
    error: "Route Not Found",
    message: `The path '${req.path}' does not exist`,
    status: 404
  });
});
```

### Benefits

✅ **User-Friendly Errors:** Instead of server crashes or null responses, clients receive a clear error message  
✅ **Prevents Confusion:** Clients know exactly why their request failed  
✅ **Better Logging:** Server can log all unmatched requests for debugging  
✅ **Professional API:** Shows you have thought about edge cases

### What NOT to Return

```json
❌ null
❌ undefined
❌ 500 Internal Server Error
❌ Silent failure (no response)

✅ Proper 404 with descriptive message
```

---

## 7. Best Practices

### 1. **Keep Routes Consistent and Semantic**
- Use **plural nouns** for collections: `/api/users`, `/api/products`
- Use **lowercase** and **hyphens** for multi-word paths: `/api/user-profiles`, not `/api/userProfiles`
- Avoid vague or ambiguous paths

```
✅ /api/users/123/posts
❌ /api/user/123/p
❌ /api/getUser/123
```

### 2. **Use HTTP Methods Correctly**
- `GET` — Fetch data (safe, idempotent)
- `POST` — Create new resources
- `PUT` — Replace entire resource
- `PATCH` — Partial update
- `DELETE` — Remove resource

```python
✅ GET /api/users          # Fetch all users
✅ POST /api/users         # Create new user
✅ GET /api/users/123      # Fetch specific user
✅ PATCH /api/users/123    # Update user 123
✅ DELETE /api/users/123   # Delete user 123

❌ GET /api/deleteUser/123
❌ POST /api/getUsers
```

### 3. **Limit Nesting to 2-3 Levels**
Too much nesting becomes hard to maintain and read:

```
✅ /api/users/123/posts/456
❌ /api/companies/1/departments/2/teams/3/members/4/projects/5
```

### 4. **Use Query Parameters for Filtering, Not Path Parameters**
```
✅ /api/posts?status=published&author=john
❌ /api/posts/published/john
```

### 5. **Order Route Definitions**
Place **more specific routes before general ones**:

```python
# ✅ Correct order
@app.get("/api/users/me")           # Specific
@app.get("/api/users/{user_id}")    # General (catches all user IDs)

# ❌ Wrong order - /users/{user_id} would catch /users/me
@app.get("/api/users/{user_id}")
@app.get("/api/users/me")
```

### 6. **Document Your Routes**
Use tools like **Swagger/OpenAPI** to auto-generate API documentation:

```python
from fastapi import FastAPI

app = FastAPI(
    title="My API",
    description="A well-documented API",
    version="1.0.0"
)

@app.get("/api/users/{user_id}", summary="Get user by ID")
def get_user(user_id: int):
    """
    Retrieve a specific user by their ID.
    
    **Parameters:**
    - user_id: The unique identifier of the user
    
    **Returns:**
    - User object with all details
    """
    return {"user_id": user_id, "name": "John"}
```

---

## 8. Real-World Examples

### Example 1: Blog API

```
GET /api/v1/posts                          # All posts
GET /api/v1/posts?page=1&limit=10          # Paginated posts
GET /api/v1/posts/123                      # Specific post
GET /api/v1/posts/123/comments             # Comments on post 123
GET /api/v1/posts/123/comments/456         # Specific comment
POST /api/v1/posts                         # Create new post
PATCH /api/v1/posts/123                    # Update post 123
DELETE /api/v1/posts/123                   # Delete post 123
POST /api/v1/posts/123/comments            # Add comment to post 123
DELETE /api/v1/posts/123/comments/456      # Delete comment
```

### Example 2: E-Commerce API

```
GET /api/v2/products                                    # All products
GET /api/v2/products?category=electronics&price_min=50 # Filtered products
GET /api/v2/products/999                                # Specific product
GET /api/v2/products/999/reviews                        # Reviews for product
GET /api/v2/products/999/reviews/5                      # Specific review
POST /api/v2/products/999/reviews                       # Create review
GET /api/v2/users/123/orders                            # User's orders
GET /api/v2/users/123/orders/456/items                  # Items in order
```

### Example 3: Social Media API

```
GET /api/users                              # All users
GET /api/users/123                          # User profile
GET /api/users/123/followers                # User's followers
GET /api/users/123/following                # Users they follow
GET /api/users/123/posts                    # User's posts
POST /api/users/123/posts                   # Create post
GET /api/users/123/posts/456/likes          # Who liked the post
POST /api/users/123/posts/456/likes         # Like a post
GET /api/users/123/notifications            # User's notifications
```

---

## Summary

| Concept | Key Takeaway |
|---------|--------------|
| **Routing** | Maps HTTP method + path → handler logic |
| **Static Routes** | Fixed paths like `/api/books` |
| **Dynamic Routes** | Paths with variables like `/api/users/:id` |
| **Path Parameters** | Identify specific resources (required) |
| **Query Parameters** | Filter, sort, paginate (optional) |
| **Nested Routing** | Express hierarchy: `/users/123/posts/456` |
| **Versioning** | Keep old & new APIs alive simultaneously |
| **Catch-All Routes** | Safety net for unmatched requests (404) |

---

## Further Reading

- [REST API Best Practices](https://restfulapi.net/)
- [HTTP Methods Explained](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
- [FastAPI Routing Documentation](https://fastapi.tiangolo.com/tutorial/first-steps/)
