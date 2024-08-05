# Using the react pre-built UI

### Install the SDK

Install our React SDK's with your favourite package manager:

`npm install @scute/react @scute/ui-react`

Add your ceredentials to your environment variable handler:

```sh
REACT_APP_SCUTE_APP_ID="YOUR_PROJECT_ID"
REACT_APP_SCUTE_BASE_URL="YOUR_BASE_URL"
```

### Initialize the Scute client

First initialize the Scute client using the `createClient` method exposed by `@scute/react` package:

```js
// scute.js

import { createClient } from "@scute/react";

export const scute = createClient({
  appId: import.meta.env.VITE_SCUTE_APP_ID as string,
  baseUrl: import.meta.env.VITE_SCUTE_BASE_URL as string,
});
```

### Wrap your React app with Scute AuthProvider

To be able to use the `useScuteCllient` and `useAuth` hooks, wrap your app inside the Scute `AuthContextProvider`:

```jsx
// App.jsx

import { AuthContextProvider } from "@scute/react";
import { scute } from "./scute";

export default function App() {
  return (
    <AuthContextProvider scuteClient={scute}>
      {/* This is where the Prebuilt component and the rest of your application lives */}
    </AuthContextProvider>
  );
}
```

### Add the Scute pre-built UI

First create the component that you would like to show to your authenticated users:

```jsx
// AuthenticatedView.jsx
import { Profile } from "@scute/ui-react";
import { useAuth } from "@scute/react";

export const AuthenticatedView = () => {
  const { session, user, signOut } = useAuth();

  if (session.status === "loading") {
    return null;
  } else if (session.status === "unauthenticated") {
    return <>unauthenticated</>;
  }

  return (
    <div style={{ margin: "1rem" }}>
      <pre style={{ fontSize: "12px" }}>
        {JSON.stringify({ user, session }, null, 4)}
      </pre>
      <Profile scuteClient={scuteClient} language="en" />
      <button onClick={() => signOut()} style={{ marginTop: "1rem" }}>
        Sign Out
      </button>
    </div>
  );
};
```

Next, create a component to switch between the authentication form and the `AuthenticatedView` based on the session status:

```jsx
// ScuteUI.jsx
import { Auth } from "@scute/ui-react";
import { useScuteClient, useAuth } from "@scute/react";
import { AuthenticatedView } from "./AuthenticatedView";

export const ScuteUI = () => {
  const { session } = useAuth();
  const scute = useScuteClient();

  if (session.status === "authenticated") {
    return <AuthenticatedView />;
  }

  return <Auth scuteClient={scute} language="en" />;
};
```

Finally, modify the `App.jsx` to render the `ScuteUI` inside the `AuthContextProvider`:

```jsx
// App.jsx

import { AuthContextProvider } from "@scute/react";
import { scute } from "./scute";
import { ScuteUI } from "./ScuteUI";

export default function App() {
  return (
    <AuthContextProvider scuteClient={scute}>
      <ScuteUI />
    </AuthContextProvider>
  );
}
```

Congrats! You have a working Scute instance now!

**No session**

![Prebuilt UI No session](/assets/no-session.png)

**With session**

![Prebuilt UI with session](/assets/with-session.png)
