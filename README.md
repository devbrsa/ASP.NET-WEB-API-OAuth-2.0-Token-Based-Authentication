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

**Configuring Web API to use Bearer Tokens:**

In the WebApiConfig.Register method, the following code sets up authentication for the Web API pipeline:

```
config.SuppressDefaultHostAuthentication();
config.Filters.Add(new HostAuthenticationFilter(OAuthDefaults.AuthenticationType));
```

The HostAuthenticationFilter class enables authentication using bearer tokens.
The SuppressDefaultHostAuthentication method tells Web API to ignore any authentication that happens before the request reaches the Web API pipeline, either by IIS or by OWIN middleware. That way, we can restrict Web API to authenticate only using bearer tokens.

When the client requests a protected resource, here is what happens in the Web API pipeline:
1. The HostAuthentication filter calls the OAuth middleware to validate the token.
2. The middleware converts the token into a claims identity.
3. At this point, the request is authenticated but not authorized.
4. The authorization filter examines the claims identity. If the claims authorize the user for that resource, the request is authorized. By default, the [Authorize] attribute will authorize any request that is authenticated. However, you can authorize by role or by other claims. For more information, see Authentication and Authorization in Web API.
5. If the previous steps are successful, the controller returns the protected resource. Otherwise, the client receives a 401 (Unauthorized) error.

![authenticationflow](https://cloud.githubusercontent.com/assets/1701237/25331121/f6f92dec-28f2-11e7-9331-6e589bcc6afe.png)

**Testing the application:**

1. Try to access protected resource using POSTMAN (google chrome extension) as anonymous user:

![access-protected-resource-anonymous-user](https://cloud.githubusercontent.com/assets/1701237/25331289/81bc6dcc-28f3-11e7-8e6d-de07bb387c8f.png)

2. Register as a new user:

![register-user](https://cloud.githubusercontent.com/assets/1701237/25331313/9ca780b8-28f3-11e7-93b8-4b822cdcbef6.png)

3. Get Token:

![get-token](https://cloud.githubusercontent.com/assets/1701237/25331340/ae924132-28f3-11e7-8529-3bb61fc86935.png)

4. Try to access protected resource using POSTMAN (google chrome extension) as registered user:

![get-protected-resource-reg-user](https://cloud.githubusercontent.com/assets/1701237/25331358/c61e8982-28f3-11e7-9f33-55ef5cddf212.png)




Main Source:
https://docs.microsoft.com/en-us/aspnet/web-api/overview/security/individual-accounts-in-web-api

