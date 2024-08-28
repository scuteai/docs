# Scute with Express JS

Scute can be integrated into an Express.js application using the `@scute/node` package. This package provides a middleware and a method called `authenticateRequest` that can be used to handle the authentication and authorization of users in your Express.js application.

Head over to the [example project repo](https://github.com/scuteai/js/tree/main/examples/with-nodejs) to run this example project and check out the [type docs](https://scute-js-docs.netlify.app/) for more `scuteClient` methods.

### Install the `@scute/node` package

First, install the `@scute/node` package using npm or yarn:

```bash
npm install @scute/node
```

### Initialize the Scute client

In your entry point (usually `app.ts`) initialize the Scute client using the `createClient` method exposed by the `@scute/node` package:

```ts
// app.ts
// ... Redacted
import {
  createClient,
  authenticateRequest,
  scuteAuthMiddleware,
  type AuthenticatedRequest,
  InvalidAuthTokenError,
} from "@scute/node";

const scute = createClient({
  appId: process.env.SCUTE_APP_ID as string,
  baseUrl: process.env.SCUTE_BASE_URL as string,
});

// ... Redacted middleware and routes
```

### Using the Scute middleware

You can use the `scuteAuthMiddleware` to protect routes that require authentication. The middleware will check if the user is authenticated and attach the user object to the request object.

```ts
// app.ts
// ... Redacted
import {
  createClient,
  authenticateRequest,
  scuteAuthMiddleware,
  type AuthenticatedRequest,
  InvalidAuthTokenError,
} from "@scute/node";

const scute = createClient({
  appId: process.env.SCUTE_APP_ID,
  baseUrl: process.env.SCUTE_BASE_URL,
});

// ... Redacted middleware and routes
app.get(
  "/authenticated-route-middleware",
  scuteAuthMiddleware(scute),
  async (req, res) => {
    const user = (req as AuthenticatedRequest).user;
    res.send(user);
  }
);
```

### Using the `authenticateRequest` helper method

You can also use the `authenticateRequest` method to authenticate the user manually in your route handlers:

```ts
// app.ts
// ... Redacted
import {
  createClient,
  authenticateRequest,
  scuteAuthMiddleware,
  type AuthenticatedRequest,
  InvalidAuthTokenError,
} from "@scute/node";

const scute = createClient({
  appId: process.env.SCUTE_APP_ID,
  baseUrl: process.env.SCUTE_BASE_URL,
});

// ... Redacted middleware and routes

app.get("/authenticated-route", async (req, res) => {
  try {
    const user = await authenticateRequest(scute, req);
    res.send(user);
  } catch (error) {
    if (error instanceof InvalidAuthTokenError) {
      res.status(401).send("Unauthorized");
    } else {
      res.status(500).send("Internal Server Error");
    }
  }
});
```
