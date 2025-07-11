---
id: basicauth
---

# BasicAuth

Basic Authentication middleware for [Fiber](https://github.com/gofiber/fiber) that provides an HTTP basic authentication. It calls the next handler for valid credentials and [401 Unauthorized](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401) or a custom response for missing or invalid credentials.

The default unauthorized response includes the header `WWW-Authenticate: Basic realm="Restricted", charset="UTF-8"` and sets `Cache-Control: no-store`.

## Signatures

```go
func New(config Config) fiber.Handler
func UsernameFromContext(c fiber.Ctx) string
func PasswordFromContext(c fiber.Ctx) string
```

## Examples

Import the middleware package that is part of the Fiber web framework

```go
import (
    "github.com/gofiber/fiber/v3"
    "github.com/gofiber/fiber/v3/middleware/basicauth"
)
```

After you initiate your Fiber app, you can use the following possibilities:

```go
// Provide a minimal config
app.Use(basicauth.New(basicauth.Config{
    Users: map[string]string{
        "john":  "doe",
        "admin": "123456",
    },
}))

// Or extend your config for customization
app.Use(basicauth.New(basicauth.Config{
    Users: map[string]string{
        "john":  "doe",
        "admin": "123456",
    },
    Realm: "Forbidden",
    Authorizer: func(user, pass string) bool {
        if user == "john" && pass == "doe" {
            return true
        }
        if user == "admin" && pass == "123456" {
            return true
        }
        return false
    },
    Unauthorized: func(c fiber.Ctx) error {
        return c.SendFile("./unauthorized.html")
    },
}))
```

Getting the username and password

```go
func handler(c fiber.Ctx) error {
    username := basicauth.UsernameFromContext(c)
    password := basicauth.PasswordFromContext(c)
    log.Printf("Username: %s Password: %s", username, password)
    return c.SendString("Hello, " + username)
}
```

## Config

| Property        | Type                        | Description                                                                                                                                                           | Default               |
|:----------------|:----------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------|
| Next            | `func(fiber.Ctx) bool`     | Next defines a function to skip this middleware when returned true.                                                                                                   | `nil`                 |
| Users           | `map[string]string`         | Users defines the allowed credentials.                                                                                                                                | `map[string]string{}` |
| Realm           | `string`                    | Realm is a string to define the realm attribute of BasicAuth. The realm identifies the system to authenticate against and can be used by clients to save credentials. | `"Restricted"`        |
| Charset         | `string`                    | Charset sent in the `WWW-Authenticate` header, so clients know how credentials are encoded. | `"UTF-8"` |
| StorePassword   | `bool`                      | Store the plaintext password in the context and retrieve it via `PasswordFromContext`. | `false` |
| Authorizer      | `func(string, string) bool` | Authorizer defines a function to check the credentials. It will be called with a username and password and is expected to return true or false to indicate approval.  | `nil`                 |
| Unauthorized    | `fiber.Handler`             | Unauthorized defines the response body for unauthorized responses.                                                                                                    | `nil`                 |

## Default Config

```go
var ConfigDefault = Config{
    Next:            nil,
    Users:           map[string]string{},
    Realm:           "Restricted",
    Charset:         "UTF-8",
    StorePassword:   false,
    Authorizer:      nil,
    Unauthorized:    nil,
}
```
