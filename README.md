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
| Mastodon Server         | Authorization Server, Resource Server |

# Choosing a Grant Type

There are four ways to authorize an app.

| Grant Type            |
|-----------------------|
| Authorization Code    |
| Implicit              |
| Resource Owner Password Credentials |
| Client Credential     |

In this documentation we only cover the first: Authorization Code

# Registration

> "The means through which the client registers
> with the authorization server are beyond the scope of this
> specification"

Use Mastodon's RESTful API to register apps. The endpoint for app registration is: 

~~~
/api/v1/apps
~~~

Your app must send a POST request with the following urlencoded data:

| Field           | Required?   | Description             |
|-----------------|-------------|-------------------------|
| `client_name`   | YES         | The name of your app    |
| `redirect_uris` | YES         | A URL your app has access to or `urn:ietf:wg:oauth:2.0:oob` URI. See details below |
| `scopes`        | YES         | See details below          |
| `website`       | NO          | URL to your app's homepage |


### redirect_uris

Same URL you will use during the authorization step below. We register it here for security reason (section 10.0).

You have two options:

1. Set this to the URL the user's web browser will request after the user has authorized or denied your app. Mastodon will append query parameters like `code` or `error` with urlencoded values.

    1. On some mobile platforms, the web browser will invoke a certain app when a certain URL Scheme is encountered. Use this to give your app access to the URL and to process the query parameter(s).
    
    2. On some platforms, the web browser will send the request to a local http server with a URL like `http://localhost`
    
    3. Some apps have servers running the URL site and can securely communicate with them

or,

2. Set this to `urn:ietf:wg:oauth:2.0:oob` if no URL should be set. Mastodon will present the user with the authorization code that the user can copy and paste into your app.

> "The redirection endpoint SHOULD require the use of TLS ...
> ... when the redirection request will result in the transmission of
> sensitive credentials over an open network."

### scopes

A scope determines what Mastodon resources your app can access.

Scopes are another one of those things left up to the server. Mastodon v2.2.0 has the following:

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

See OAUth spec section 3.3 for scopes

# Authorization




