# Disclaimer

I MAKE NO WARRANTIES REGARDING THE USE OR ACCURACY OF THIS DOCUMENT OR THE INFORMATION PROVIDED HEREUNDER, AND DISCLAIM LIABILITY FOR DAMAGES RESULTING FROM THE USE OF THIS DOCUMENT OR THE INFORMATION PROVIDED HEREUNDER. 

# Source

The OAuth 2.0 specification is described in [RFC6749 The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)

# Terms

Because of its intended wide use the specification uses broad terms. But for our use here we use less formal terms.


| This documentation uses | OAuth documentation uses |
|-------------------------|--------------------------|
| app                     | client                   |
| user                    | resource owner           |
| (web) browser           | user agent               |


# Registration

> [The OAuth 2] specification leaves a few required components partially or fully undefined (e.g., client registration, ...

Use Mastodon's RESTful API to register apps. The endpoint for app registration is: 

/api/v1/apps

Your app must send a POST request with the following urlencoded data:

| Field         | Required?   | Description             |
|---------------|-------------|-------------------------|
| client_name   | YES         | The name of your app    |
| redirect_uris | YES         | see details             |
| scopes        | YES         | see details             |
| website       | NO          | URL to your app's homepage |


### redirect_uris

### scopes

A scope determines what Mastodon resources your app can access.

Scopes are another one of those things left up to the server. Mastodon v2.2.0 has the following:

| Scope  | Description |
|--------|-------------|
| read   | read data   |
| write  | post statuses and upload media for statuses |
| follow | follow, unfollow, block, unblock |

* multiple scopes need to be space separated
* scopes are case-sensitive
* order not relevant

Mastodon does not provide default scopes

> Some resources require no authorization or scope. See Mastodon's API documentation

See OAUth spec section 3.3 for scopes

# Authorization

There are for ways to authorize an app.

| Grant Type            |
|-----------------------|
| Authorization Code    |
| Implicit              |
| Resource Owner Password Credentials |
| Client Credential     |

In this documentation we cover the first **grant type**: Authorization Code


