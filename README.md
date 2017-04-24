### ASP.NET WEB API OAuth 2.0 Token Based Authentication

**What is token:**

Access token is piece of data which is created by server, and used to identify the certain user of given application, and it is used to access particular resource on the server.

**Bearer Token Type:**

The access token type provides the client with the information required to successfully utilize the access token to make a protected resource request (along with type-specific attributes).  The client MUST NOT use an access token if it does not understand the token type. In this example we are using token of type "Bearer" A certain type of token, with the property that anyone can use the token, and it is commonly used. Bearer can be simply understood as "give access to the Bearer of this token." It is recommended to use Bearer token over https, with short expiration time.

**Why token based authentication instead of cookie based:**

Cookies:
* Sent with every request 
* Usually supported in browsers
* Difficult to use cross domain
* Prone to CSRF

Token:
* Can be used by hetrogneous clients (browsers, native mobile app etc.)
* Work cross domain
* Scalable (no overhead in using web farm when new server is added)
* Offer more control
* Loosly Coupled

**Demo Application (Server Side/Back End):**

- Open VS 2017
- File -> Project -> Web -> ASP.NET Web Application

![project-tempalate-1](https://cloud.githubusercontent.com/assets/1701237/25327600/c44a6dc8-28e5-11e7-8221-21c16e261db2.PNG)

![project-tempalate-2](https://cloud.githubusercontent.com/assets/1701237/25327619/dbee34fa-28e5-11e7-94d1-a7829257602a.PNG)

**Authorization Server Configuration:**

* App_Start-> startup.cs partial class has the configration code as following:

```
PublicClientId = "self";
OAuthOptions = new OAuthAuthorizationServerOptions
{
    TokenEndpointPath = new PathString("/Token"),
    Provider = new ApplicationOAuthProvider(PublicClientId),
    AuthorizeEndpointPath = new PathString("/api/Account/ExternalLogin"),
    AccessTokenExpireTimeSpan = TimeSpan.FromDays(14),
    // Note: Remove the following line before you deploy to production:
    AllowInsecureHttp = true
}

// Enable the application to use bearer tokens to authenticate users
app.UseOAuthBearerTokens(OAuthOptions);

```

The TokenEndpointPath property is the URL path to the authorization server endpoint. That's the URL that app uses to get the bearer tokens.

The Provider property specifies a provider that plugs into the OWIN middleware, and processes events raised by the middleware.

Here is the basic flow when the app wants to get a token:

1. To get an access token, the app sends a request to ~/Token.
2. The OAuth middleware calls GrantResourceOwnerCredentials on the provider.
3. The provider calls the ApplicationUserManager to validate the credentials and create a claims identity.
4. If that succeeds, the provider creates an authentication ticket, which is used to generate the token.


![gettokenflow](https://cloud.githubusercontent.com/assets/1701237/25328088/918eed44-28e7-11e7-9a0e-6d419204a0ce.png)

The OAuth middleware doesn't know anything about the user accounts. The provider communicates between the middleware and ASP.NET Identity. For more information about implementing the authorization server.

