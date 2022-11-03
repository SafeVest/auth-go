# gotrue-go

![example branch parameter](https://github.com/kwoodhouse93/gotrue-go/actions/workflows/test.yaml/badge.svg?branch=main)
[![codecov](https://codecov.io/gh/kwoodhouse93/gotrue-go/branch/main/graph/badge.svg?token=JQQJKETMRX)](https://codecov.io/gh/kwoodhouse93/gotrue-go)
![GitHub](https://img.shields.io/github/license/kwoodhouse93/gotrue-go)
[![Go Reference](https://pkg.go.dev/badge/github.com/kwoodhouse93/gotrue-go.svg)](https://pkg.go.dev/github.com/kwoodhouse93/gotrue-go)


A Golang client library for the [Supabase GoTrue](https://github.com/supabase/gotrue) API.

> ⚠️ Using [`netlify/gotrue`](https://github.com/netlify/gotrue)?
>
> The types in this library assume you are interacting with a Supabase GoTrue server. It is very unlikely to work with a Netlify GoTrue server.

For more information about the Supabase fork of GoTrue, [check out the project here](https://github.com/supabase/gotrue).

## Project status

This library is a pre-release work in progress. It has not been thoroughly tested, and the API may be subject to breaking changes, and so it should not be used in production.

Required for V1 release:
- Implement and test endpoints
    - Client API
        - [X] GET /health
        - [X] GET /settings
        - [ ] GET /callback
        - [ ] POST /callback
        - [ ] GET /authorize
        - [ ] POST /invite
        - [ ] POST /signup
        - [ ] POST /recover
        - [ ] POST /magiclink
        - [ ] POST /otp
        - [ ] POST /token
        - [ ] GET /verify
        - [ ] POST /verify
        - [ ] POST /logout
        - [ ] GET /reauthenticate
        - [ ] GET /user
        - [ ] PUT /user
        - [ ] POST /factors
        - [ ] POST /factors/{factor_id}/verify
        - [ ] POST /factors/{factor_id}/challenge
        - [ ] DELETE /factors/{factor_id}
        - [ ] GET /sso/saml/metadata
        - [ ] POST /sso/saml/acs
    - Admin API
        - [ ] GET /admin/audit
        - [X] GET /admin/users
        - [X] POST /admin/users
        - [ ] GET /admin/users/{user_id}/factors
        - [ ] DELETE /admin/users/{user_id}/factors/{factor_id}
        - [ ] PUT /admin/users/{user_id}/factors/{factor_id}
        - [ ] GET /admin/users/{user_id}
        - [ ] PUT /admin/users/{user_id}
        - [ ] DELETE /admin/users/{user_id}
        - [ ] POST /admin/generate_link
        - [ ] GET /admin/sso/providers
        - [ ] POST /admin/sso/providers
        - [ ] GET /admin/sso/providers/{idp_id}
        - [ ] PUT /admin/sso/providers/{idp_id}
        - [ ] DELETE /admin/sso/providers/{idp_id}
- Test infrastructure
    - [X] Postgres container with GoTrue config
    - [X] GoTrue container - signup enabled, autoconfirm off
    - [ ] GoTrue container - signup enabled, autoconfirm on
    - [ ] GoTrue container - signup disabled
    - [ ] Mail server

## Quick start

### Install

```sh
go get github.com/kwoodhouse93/gotrue-go
```

### Usage
```go
package main

import "github.com/kwoodhouse93/gotrue-go"

const (
    projectReference = "<your_supabase_project_reference>"
    apiKey = "<your_supabase_anon_key>"
)

func main() {
    // Initialise client
    client := gotrue.New(
        projectReference,
        apiKey,
    )

    // Log in a user (get access and refresh tokens)
    resp, err := client.Token(gotrue.TokenRequest{
        GrantType: "password",
        Email: "<user_email>",
        Password: "<user_password>",
    })
    if err != nil {
        log.Fatal(err.Error())
    }
    log.Printf("%+v", resp)
}
```

## Options

The client can be customized with the options below.

In all cases, **these functions return a copy of the client**. To use the configured value, you must use the returned client. For example:

```go
client := gotrue.New(
    projectRefernce,
    apiKey,
)

token, err := client.Token(gotrue.TokenRequest{
        GrantType: "password",
        Email: email,
        Password: password,
})
if err != nil {
    // Handle error...
}

authedClient := client.WithToken(
    token.AccessToken,
)
user, err := authedClient.GetUser()
if err != nil {
    // Handle error...
}
```

### WithToken
```go
func (*Client) WithToken(token string) *Client
```

Returns a client that will use the provided token in the `Authorization` header on all requests.

### WithCustomGoTrueURL
```go
func (*Client) WithCustomGoTrueURL(url string) *Client
```

Returns a client that will use the provided URL instead of `https://<project_ref>.supabase.com/auth/v1/`. This allows you to use the client with your own deployment of the GoTrue server without relying on a Supabase-hosted project.

### WithClient
```go
func (*Client) WithClient(client http.Client) *Client
```

By default, the library uses a default http.Client. If you want to configure your own, pass one in using `WithClient` and it will be used for all requests made with the returned `*gotrue.Client`.

## Testing

> You don't need to know this stuff to use the library

The library is tested against a real GoTrue server running in a docker image. This also requires a postgres server to back it. These are configured using docker compose.

To run these tests, simply `make test`.

To interact with docker compose, you can also use `make up` and `make down`.
