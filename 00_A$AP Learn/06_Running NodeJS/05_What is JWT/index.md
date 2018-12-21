### What is JWT, Theory Aside

Literally, JWT is a `JSON Web Token`. The theory behind it has better be right, as this is cornerstone security access feature in many modern multi-server apps, including the one in the demo - although, in the demo, we don't actually implement this feature in the course (you can see a full implementation of JWT in the `https://github.com/exlskills` server components, e.g., `https://github.com/exlskills/auth-server`)

The idea behind JWT is that the client, upon login via some dedicated authentication endpoint, receives a secure *token* that it passes along with every request to other endpoints in scope, regardless if those endpoints are on the same or different servers (remember, those servers should have `cors` configured too). The key feature of the JWT approach is that the servers do not check with the authentication endpoint or any other system/database to see whether the token is valid and its possessor legitimate. All the information that the servers need is securely encoded inside the JWT (incidently, in the JSON format - here's the "J"). It's the responsibility of the authentication endpoint to put together the content of the JWT, upon successful authentication, of the *client*, and encode it into the JWT it sends to the client.

The JWT is passed from the client to the servers in HTTPS, in the cookies, so no one else can get a hold of it, and the client cannot alter the JWT's content as it has no *key* to either read or write into the token. The JWT-decoding keys are present on each server in scope. Here's the `src/config/sample_keys` folder we've seen in the project structure.

When the client terminates the session, the cookie with the JWT is deleted. The server, technically, cannot efficiently "revoke" the JWT as if the client saves it somewhere, it may succeed recreating the cookie without obtaining a new JWT. So, the assumption on the server side should be that once the JWT has been issued - it will remain valid for as long as it is intended to be valid - no early revocation.

So, in the User-Order use case, the User is first directed to the *login* component in the servers set. For better design and added security, that component should be isolated on a dedicated server, not here on the GraphQL one. The commented out `middleware.loginRequired` function checks for the presence of the JWT and redirects if it is not.

On the authentication endpoint, the User goes through the login steps, the JWT is encoded with the User's info, passed to it in a cookie, and the User is redirected back to the GraphQL server (or where ever the User was before being sent for the authentication). Technically, on the GraphQL server we would not need to keep any potentially confidential User info such as email, or may be anything at all (!). The JWT will have the User ID that we read into the `viewer` object being used everywhere in the GraphQL server flow.

So, consider the User Collection being added for demo purposes only - you will likely not need it in a real multi-server design.


Next - logging. Without it we're dead in the water developing, testing, troubleshooting, monitoring. So, logging is the key. Let's get it reviewed - you'll see it is very simple here!