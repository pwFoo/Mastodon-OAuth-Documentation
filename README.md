# Disclaimer

I MAKE NO WARRANTIES REGARDING THE USE OR ACCURACY OF THIS DOCUMENT OR THE INFORMATION PROVIDED HEREUNDER, AND DISCLAIM LIABILITY FOR DAMAGES RESULTING FROM THE USE OF THIS DOCUMENT OR THE INFORMATION PROVIDED HEREUNDER. 

# Reference

The OAuth 2.0 specification is described in [RFC6749 The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)

# Terms

Because of its intended wide use the specification uses broad terms. But for our use here we use less formal terms.


| This documentation uses | OAuth documentation uses |
|-------------------------|--------------------------|
| app                     | client                   |
| user                    | resource owner           |
| (web) browser           | user agent               |
| server                  | Authorization Server, Resource Server |

# Choosing a Grant Type

There are four ways to authorize an app.

| Grant Type            |
|-----------------------|
| Authorization Code    |
| Implicit              |
| Resource Owner Password Credentials |
| Client Credential     |

In this documentation we only cover the first: Authorization Code

# Registering an App

> "The means through which the client registers
> with the authorization server are beyond the scope of this
> specification"

Use Mastodon's RESTful API to register apps. The endpoint for app registration is: 

~~~
/api/v1/apps
~~~

Your app must send a POST request with the following url-encoded data:

| Field           | Required?   | Description             |
|-----------------|-------------|-------------------------|
| `client_name`   | YES         | The name of your app    |
| `redirect_uris` | YES         | A URL your app has access to or `urn:ietf:wg:oauth:2.0:oob` URI. See details below |
| `scopes`        | YES         | See details below          |
| `website`       | NO          | URL to your app's homepage |


Server returns JSON data. See Registration Response below.

### redirect_uris

Same URL you will use during the authorization step below. We register it here for security reasons (see section 10.0).

You have two options:

1. Set this to the URL the user's browser will request after your app has been authorized (or denied). Mastodon will append query parameters like `code` or `error` with url-encoded values.

    1. On some mobile platforms, the web browser will invoke a certain app when a certain URL Scheme is encountered. Use this to give your app access to the URL and to process the query parameter(s).
    
    2. On some platforms, the web browser will send the request to a local http server when supplied a URL like `http://localhost`
    
    3. Some apps have servers running the URL site and can securely communicate with them

or

2. Set this to `urn:ietf:wg:oauth:2.0:oob` if no URL should be set. Mastodon will present the user with the authorization code that the user can copy and paste into your app.

> "The redirection endpoint SHOULD require the use of TLS ...
> when the redirection request will result in the transmission of
> sensitive credentials over an open network."

Note care should also be taken when copying code to a shared clipboard.

TODO - can multiple URIs be registered? if so, how? space delimited?

### scopes

A scope determines what Mastodon resources your app can access.

Scopes are another one of those things left up to the server. Mastodon v2.2.0 has the following scopes:

| Scope   | Description |
|---------|-------------|
| `read`  | read data   |
| `write` | post statuses and upload media for statuses |
| `follow`| follow, unfollow, block, unblock |

* multiple scopes need to be space separated
* scopes are case-sensitive
* order not relevant

Mastodon does not provide default scopes

> Some resources require no authorization or scope. See Mastodon's API documentation

See OAUth spec section 3.3 for more about scopes

### Registration Response

| Field          | Description |
|----------------|-------------|
| `id`           |             |
| `client_id`    | public information used to identify your app during authorization. |
| `client_secret`| confidential information used to identify your app when requesting a token. |


# Authorizing an App

OAuth 2.0 utilizes two server end points:

| Enpoint                   | Description | On Mastodon |
|---------------------------|-------------|-------------|
| Authorization Endpoint    | Used by the app to obtain authorization from a user | `/oauth/authorize` |
| Token Endpoint            | used by the app to exchange an authorization grant (e.g. authorization code) for an access token | `/ouath/token` |


Again, the actual URL to these endpoints are left up to the server.


Use Mastodon's RESTful API to authorize apps. The endpoint for app authorization is: 

~~~
/oauth/authorize
~~~

Your app must direct an external or internal web browser using the following url-encoded query parameters:

| Field           | Required?   | Description             |
|-----------------|-------------|-------------------------|
| `response_type` | YES         | `code` for Authorization Code grant type  |
| `client_id`     | YES         | A string given to you during registration |
| `redirect_uri`  | MAYBE       | same as registration. For additional notes, see details below. |
| `scope`         | YES         | same as registration. See registration for details. |
| `state`         | NO          | recommended if combatting CSRF is necessary. see 10.12 |

parameters with no values are ignored. Unrecognized parameters are ignored. Parameters must not be included more than once.

TODO - Servers may support POST request. Does Mastodon?
TODO - can redirect be opt as described in 3.1.2
TODO - must scope be the same

### redirect_uri

If invalid, or mismatched, request will fail.

The app should not include any untrusted 3rd party scripts in the URL because any script included in the HTML document will execute with full access to the URI

TLS is recommended if the GET request made by the web browser for the URL travels across an insecure or open network.

> Mastodon uses [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper/wiki) to provide its OAuth 2.0 features. The Doorkeeper documentation can be a helpful companion.

See Registration section above for further details.

### Authorization Response

At this point, the web browser will direct the user to a page on the Mastodon server to authorize your app. If the user is not signed in, the user will be asked to sign in. Your app must be registered with that server.

If your request is granted, the server will either:

1. direct the web browser to the URL you gave it, adding a `code` query parameter. A `state` parameter will be included if one was included in the request.

2. or if `urn:ietf:wg:oauth:2.0:oob` was given, to a page with the authorization code where the user can copy it.

### Server Responded with Authorization Code

OAuth requires the Authorization Code to expire. It recommends 10 minutes. 

Your app must not reuse one. Reusing an authorization code will cause the server to deny your request, but could also cause the server to revoke tokens previously issued using that authorization code.

An authorization code is also associated with a client ID and redirect URI.

TODO - how long do auth code last
TODO - does server revoke tokens prev assigned based on reused auth code

### Server Responded with an Error

If the user or server denied your request, or an error occurred, the web browser is set to the URL you gave it with `error` and `error_description` parameters added. A `state` parameter will be included if one was included in the request.

The value assigned to the error parameter may be one of the following

| Error Code        |
|-------------------|
| `invalid_request` |
| `unauthorized_client` |
| `access_denied`   |
| `invalid_scope`   |
| `server_error`    |
| `temporarily_unavailable` |

Mastodon does not supply an optional `error_uri` parameter.

# Requesting an Access Token

Use Mastodon's RESTful API to request an OAuth access token. The endpoint for token requests is: 

~~~
/oauth/token
~~~

Your app must use POST to send the following data

| Field           | Required?   | Description             |
|-----------------|-------------|-------------------------|
| `grant_type`    | YES         | must be `authorization_code`    |
| `code`          | YES         | used to supply the authorization code |
| `redirect_uri`  | MAYBE       | same as registration. See registration for details. |
| `client_id`     | YES         | obtained during registration. See registration for details. |
| `client_secret` | YES         | obtained during registration. |

parameters with no values are ignored. Unrecognized parameters are ignored. Parameters must not be included more than once.

### Server Responded with Access Token

The following parameters are included in the body of the HTTP response using the `application/json` media type.

| Field          | Description |
|----------------|-------------|
| `access_token` | a string of unspecified length used to request specific resources |
| `token_type`   | 'bearer' or 'mac'. Always 'bearer' |
| `scope`        | scope of the issued token. may differ from the one requested during authorization. optional if identical but always included in Mastodon |
| `created_at`   |  possibly an undocumented value. per the spec, 'an app must ignore unknown value names in the response' |           |

The following optional parameters are not included: `expires_in` and `refresh_token`. When `expires_in` is omitted the server should provide the expiration time via other means or document the default value.

TODO - what is the default lifetime of a token

### Server Responded with an Error

Server responds with 400 and includes the following content

| Field                 | Description                   |
|-----------------------|-------------------------------|
| `error`               |  an error code. see below     |
| `error_description`   |  a human readable description |

error is assigned one of the following strings

| Error Code        |
|-------------------|
| `invalid_request` |
| `invalid_client`  |
| `invalid_grant`   |
| `unauthorized_client` |
| `unsupported_grant_type`   |
| `invalid_scope`   |

Mastodon does not supply an optional `error_uri` parameter.

# Refreshing an Access Token

It appears Mastodon no longer uses a refresh token for apps authorized using the Authorization Code grant type.

# Accessing a Resource Using an Access Token

# Security

See the specification and the its references for details
