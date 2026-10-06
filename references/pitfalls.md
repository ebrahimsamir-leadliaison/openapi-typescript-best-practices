# Common Pitfalls

## ❌ 1. Missing `operationId`

Without `operationId`, generated method names will be generic:

```json
{
  "paths": {
    "/users": {
      "get": {
        "summary": "Get all users"
        // ❌ Missing operationId
      }
    }
  }
}
```

✅ **ALWAYS add `operationId`:**
```json
{
  "paths": {
    "/users": {
      "get": {
        "operationId": "getUsers",
        "summary": "Get all users"
      }
    }
  }
}
```

## ❌ 2. Inconsistent Property Casing

```json
{
  "properties": {
    "firstName": { "type": "string" },
    "last_name": { "type": "string" },  // ❌ Inconsistent
    "PhoneNumber": { "type": "string" }  // ❌ Wrong case
  }
}
```

✅ **Be consistent - use camelCase:**
```json
{
  "properties": {
    "firstName": { "type": "string" },
    "lastName": { "type": "string" },
    "phoneNumber": { "type": "string" }
  }
}
```

## ❌ 3. Generic Schema Names

```json
{
  "Request": { ... },    // ❌ Too generic
  "Response": { ... },   // ❌ Too generic
  "Data": { ... }        // ❌ Too generic
}
```

✅ **Use specific, descriptive names:**
```json
{
  "CreateUserRequest": { ... },
  "UserResponse": { ... },
  "UserAnalyticsData": { ... }
}
```

## ❌ 4. Overusing `additionalProperties`

```json
{
  "User": {
    "type": "object",
    "properties": {
      "id": { "type": "integer" }
    },
    "additionalProperties": true  // ❌ Defeats type safety
  }
}
```

**Result:**
```typescript
export interface User {
  id?: number;
  [key: string]: any;  // ❌ Any property allowed
}
```

✅ **Define all properties explicitly:**
```json
{
  "User": {
    "type": "object",
    "properties": {
      "id": { "type": "integer" },
      "metadata": {
        "type": "object",
        "additionalProperties": { "type": "string" }  // ✅ Typed map
      }
    }
  }
}
```

## ❌ 5. Missing Response Schemas

```json
{
  "responses": {
    "200": {
      "description": "Success"
      // ❌ No schema defined
    }
  }
}
```

✅ **ALWAYS define response schemas:**
```json
{
  "responses": {
    "200": {
      "description": "User retrieved successfully",
      "content": {
        "application/json": {
          "schema": {
            "$ref": "#/components/schemas/User"
          }
        }
      }
    }
  }
}
```

## ❌ 6. Missing or Fabricated Body for No-Input Endpoints

```json
{
  "paths": {
    "/auth": {
      "post": {
        "operationId": "authenticate"
        // ❌ requestBody omitted even though the client sends `{}`
      }
    }
  }
}
```

✅ **Model the empty body with a reusable schema:**
```json
{
  "components": {
    "schemas": {
      "EmptyRequestBody": {
        "type": "object",
        "additionalProperties": false,
        "maxProperties": 0,
        "example": {}
      }
    }
  },
  "paths": {
    "/auth": {
      "post": {
        "operationId": "authenticate",
        "requestBody": {
          "required": false,
          "content": {
            "application/json": {
              "schema": { "$ref": "#/components/schemas/EmptyRequestBody" }
            }
          }
        }
      }
    }
  }
}
```
**Why:** documents that a body isn't required but must be empty if sent, instead of silently accepting arbitrary ignored fields.

