# bailsman-api documentation


## 1. Authentication

* ### General flow
For landlord authentication OAuth 2.0 protocol is used.
To describe the process, we'll use this diagram taken from RFC 6749 (the official word on OAuth):

```
  +--------+                                           +---------------+
  |        |--(A)------- Authorization Grant --------->|               |
  |        |                                           |               |
  |        |<-(B)----------- Access Token -------------|               |
  |        |               & Refresh Token             |               |
  |        |                                           |               |
  |        |                            +----------+   |               |
  |        |--(C)---- Access Token ---->|          |   |               |
  |        |                            |          |   |               |
  |        |<-(D)- Protected Resource --| Resource |   | Authorization |
  | Client |                            |  Server  |   |     Server    |
  |        |--(E)---- Access Token ---->|          |   |               |
  |        |                            |          |   |               |
  |        |<-(F)- Invalid Token Error -|          |   |               |
  |        |                            +----------+   |               |
  |        |                                           |               |
  |        |--(G)----------- Refresh Token ----------->|               |
  |        |                                           |               |
  |        |<-(H)----------- Access Token -------------|               |
  +--------+           & Optional Refresh Token        +---------------+
```

In step A, we ask the authorization server to grant authorization to our client. The authorization server should prompt the user for recognition of this application; if granted, then access and refresh tokens are issued. The access token has a time-to-live value associated with it.

When downloading protected resources (steps C and D), the access token must be presented. If an expired or invalid token is presented, it will be rejected (steps E and F). Then you should attempt a refresh (step G.) If the refresh fails, you should start over from step A.

* ### Bailsman base URL

Bailsman base URL:
- production environment (https://bailsman.co)
- demo environment (https://bailsman-demo-app.herokuapp.com)

* ### Credentials for your application

The first step is to register your client app. Ask Bailsman contact person for client ID and a client secret for your organization. These two numbers will be used in your app, so store them as appropriate. 

* ### Requesting the Grant

To request the authorization token, you should visit the `/oauth/authorize` endpoint. In order to prompt the user to grant the access, you need to open a browser window. That window will open `BASE_URL/oauth/authorize?client_id=XXX&response_type=code&redirect_uri=YYY` with the following stand-in values:

- XXX stands for the client ID received in the previous step.
- YYY is the callback URI you configured in the previous step. These must match exactly or your request will be rejected.

When the user grants access to your application, that callback URI will be triggered and you will receive authorization code. 

* ### Getting an Access Token

To request the access token, you should use the returned code and exchange it for an access token. You need to make `POST` request to `BASE_URL/oauth/token`. The required parameters must be passed as form-urlencoded. The required parameters are:
- `client_id`: Your client ID.
- `client_secret`: Your client secret.
- `redirect_uri`: The callback URI.
- `grant_type`: This is `authorization_code`.
- `code`: The authorization code received in the previous step.

The response body that comes back looks like this:
```
{
    "access_token": "BLTl6Oo3s6L8tKnL1XJcKTaKPwF....",
    "token_type": "Bearer",
    "expires_in": 7200,
    "refresh_token": "Vz26qmPy99RKWp_nFFBXm9T4IhtP0n5M7skIdRFrs94",
    "scope": "write",
    "created_at": 1636019770
}
```

You can now make requests to the API with the access token returned. 
The `access_token` is what is needed to set in all HTTP headers; it will expire in `expires_in` seconds. Once that period has passed, you should use the `refresh_token` to get a new `access_token`. To keep from re-authorizing each time the app runs, the `refresh_token` should be persisted somewhere.

* ### Using the Access Token

For each request to access a protected resource, you will want to add this HTTP header:
`Authorization: Bearer BLTl6Oo3s6L8tKnL1XJcKTaKPwF...`
And thus the access will be granted.
Access token expiration time is 2 hours. Then you need to refresh access token. 

* ### Refresh the Access Token

When you need to refresh access token, you should make a `POST` request very similar to the one following the authorization grant (`BASE_URL/oauth/token`). Only a few values are different, marked in bold:

- `client_id`: The client ID issued in the first step.
- `client_secret`: The client secret issued in the first step.
- `redirect_uri`: The callback URI you defined previously.
- **`grant_type`: This was authorization_code in the initial request, but now it'll be `refresh_token`.**
- **`refresh_token`: The saved refresh token.**

You will get new access token for landlord in the response.

* ### Revoking Access

Post here with client credentials (in basic auth or in params client_id and client_secret) to revoke an access token. So, you need to make `POST` request to `BASE_URL/oauth/revoke` with params:
```
client_id:     9b36d8c0db5... (Your client ID)
client_secret: d6ea2770395... (Your client secret)
token:         dbaf9757982... (user access token)
```

The response body that comes back looks like this:
`{}`

## 2. Endpoints

You can find available endpoints here:

https://bailsman-demo-app.herokuapp.com/documentation
