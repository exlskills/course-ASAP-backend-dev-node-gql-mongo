### What is JWT, Theory Aside

JWT stands for `JSON Web Token`. 

JWT is an Internet Engineering Task Force (IETF) standard and it is better be good, as it's a cornerstone security access feature in many modern multi-server apps, including the demo project. Although, in the demo project the functionality is commented out. To see a similar working implementation of JWT, check out `https://github.com/exlskills` server components, `https://github.com/exlskills/auth-server` specifically.

Like the wast majority of auth and security stuff, JWT relies on the use of public/private key pairs. JWT is created by the possessor of the *private* key and used by those who have the *public* one. Here comes the presence of the *public* key in the demo `src/config/sample_keys` folder - for the use by the GraphQL server.

In the JWT process flow, the browser client, upon logging in via some dedicated authentication endpoint, receives a secure *token* created by the endpoint specifically for the client's session. The client then has to attach the token to every request to all endpoints in scope of the backend system. Regardless if the endpoints are on the same or different servers as the authentication endpoint, by reading the token they know that the client has authenticated and who the client is. 

A groundbreaking convenience feature of the JWT approach is that the servers do not have to check with the authentication endpoint or any other system/database to confirm that the client is legitimate and get its identity. All the information that the servers need about the client or the session is securely encoded inside the JWT (incidently, in the JSON format - here's the "J"). It's the responsibility of the authentication endpoint to put together the content of the JWT, upon successful authentication of the *client*, and encode it into the JWT being provided to the client.

The JWT is passed from the client to the servers in HTTPS, in the cookies, so no one else can get a hold of it, and the client cannot alter the JWT's content as it has no *private key* required to generate a valid token. Likewise, the client would not be able to read a fully encoded token as it doesn't have the *public key* either - only the backend servers in scope do.

When the client terminates the session, the cookie with the JWT is deleted. The server, technically, cannot efficiently "revoke" the JWT as if the client saves it somewhere, it may succeed recreating the cookie without obtaining a new JWT. So, the assumption on the server side should be that once the JWT has been issued - it will remain valid for as long as it is intended to be valid - no early revocation.

The IETF has everything about the token detailed in the [Standard](https://tools.ietf.org/html/rfc7519). If you get a kick from reading legal documents or Tax Form instructions - you'd enjoy reviewing the standard. Otherwise, just pick a 3rd party package that can handle the token flow for you and brief yourself on its instructions. `jsonwebtoken` is the one used in the demo project.


### JWT (Intended) Use in The Demo Project

So, in the User-Order demo project Use Case, the User is first directed to the *login* component configured on one of the backend servers (not implemented in the demo project). For better design and added security, that component should be isolated in a protected location. On the GraphQL server, the commented out `middleware.loginRequired` function checks for the presence of the JWT and redirects to the authentication endpoint if JWT is not present.

On the authentication endpoint (not implemented in the demo project), the User completes login steps, and the JWT is created with the User's info and passed to the User's browser in a cookie. The User is then redirected back to the GraphQL server. In this scenario, there is no need to keep any confidential User info in the GraphQL server database, such as email address, or any User info at all (!). The JWT may have the User info required for processing encoded in it.

Further in the GraphQL server middleware flow, User ID is decoded from the JWT and put into the `viewer` object.

Basically, JWT makes most of the MongoDB User Collection obsolete: runtime-required User info can be passed in the JWT, so only the data used in post- or analytical processing should be kept in the GraphQL server. Lots of room to optimize the design and protect your backend from break-ins exposing confidential User data.


Next - logging. Without it we're dead in the water developing, testing, troubleshooting, monitoring. So, logging is the key. Let's get it reviewed - you'll see it is very simple here!