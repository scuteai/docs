# Using the react pre-built UI

### Install the SDK

Install our React SDKs with your favorite package manager:

`npm install @scute/react @scute/ui-react`

Add your credentials to your environment variable handler:

```sh
VITE_SCUTE_APP_ID="YOUR_PROJECT_ID"
VITE_SCUTE_BASE_URL="YOUR_BASE_URL"
```

**NOTE**: If you are not using Vite, use "REACT_APP" as your prefix for your environment variables.

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

### Wrap your React app with Scute AuthContextProvider

To be able to use the `useScuteClient` and `useAuth` hooks, wrap your app inside the Scute `AuthContextProvider`:

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

First, create a component to show to your authenticated users:

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
      <Profile scuteClient={scuteClient} language="en" />
    </div>
  );
};
```

Then, create a component to switch between the authentication form and the `AuthenticatedView` based on the session status:

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

  return (
    <Auth
      scuteClient={scute}
      language="en"
      policyURLs={{
        privacyPolicy: "https://example.com/privacy",
        termsOfService: "https://example.com/terms",
      }}
      logoUrl={themes[index].logoUrl}
    />
  );
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

## Component API's

### `Auth` Component

| Property    | Type                                                | Default    | Description                                          |
| ----------- | --------------------------------------------------- | ---------- | ---------------------------------------------------- |
| scuteClient | ScuteClient                                         | undefined  | The Scute client instance. This property is required |
| onSignIn?   | () => void                                          | undefined  | Callback function for sign-in                        |
| webauthn?   | "strict" &#124; "optional" &#124; "disabled"        | "optional" | Options for WebAuthn                                 |
| language?   | string                                              | "en"       | Language setting                                     |
| appearance? | { theme?: [Theme](#theme-object) }                  | undefined  | Appearance settings                                  |
| policyURLs? | { privacyPolicy?: string; termsOfService?: string } | undefined  | URLs for policy documents                            |
| logoUrl?    | string                                              | undefined  | URL for the application logo                         |

### `Profile` Component

| Property    | Type                               | Default   | Description               |
| ----------- | ---------------------------------- | --------- | ------------------------- |
| scuteClient | ScuteClient                        | undefined | The Scute client instance |
| language?   | string                             | "en"      | Language setting          |
| appearance? | { theme?: [Theme](#theme-object) } | undefined | Appearance settings       |

### `Theme` object

Theme object has a colors property to change the default colors for the pre-built component. Let's say you wanted to make the primary button pink:

```jsx
<Auth
  scuteClient={scuteClient}
  language="en"
  appearance={{ theme: { colors: { buttonIdleBg: "pink" } } }}
/>
```

Here is a full list of colors and their default values:

| Property                | Default Value             |
| ----------------------- | ------------------------- |
| errorColor              | `#fe4f0d`                 |
| svgIconColor            | `#121212`                 |
| svgIconMutedColor       | `#f7f7f7`                 |
| loadingSpinnerColor     | `#cccccc`                 |
| loadingSpinnerBorder    | `#f1f1f1`                 |
| surfaceBg               | `#f7f7f7`                 |
| surfaceLink             | `#666666`                 |
| surfaceText             | `#666666`                 |
| surfaceTextBg           | `#ffffff`                 |
| cardBg                  | `#ffffff`                 |
| cardHeadingText         | `#121212`                 |
| cardBodyText            | `#121212`                 |
| cardFooterText          | `#b0b0b0`                 |
| cardFooterLink          | `#b0b0b0`                 |
| panelBg                 | `#f7f7f7`                 |
| panelText               | `#121212`                 |
| inputBg                 | `#ffffff`                 |
| inputText               | `#121212`                 |
| inputPlaceholder        | `#333333`                 |
| inputBorder             | `#333333`                 |
| inputFocusGlow          | `rgba(46, 234, 175, 0.3)` |
| inputDisabledBg         | `#dedede`                 |
| inputDisabledText       | `#333333`                 |
| buttonIconColor         | `#ffffff`                 |
| buttonIdleText          | `#ffffff`                 |
| buttonIdleBg            | `#212121`                 |
| buttonIdleBorder        | `transparent`             |
| buttonIdleShadow        | `transparent`             |
| buttonPassiveBg         | `#f7f7f7`                 |
| buttonPassiveText       | `#bababa`                 |
| buttonHoverBg           | `#121212`                 |
| buttonHoverText         | `#ffffff`                 |
| buttonHoverBorder       | `black`                   |
| buttonHoverShadow       | `transparent`             |
| buttonFocusBorder       | `rgba(46, 234, 175, 0.3)` |
| buttonFocusShadow       | `rgba(46, 234, 175, 0.3)` |
| buttonAltIconColor      | `#ffffff`                 |
| buttonAltIdleText       | `#333333`                 |
| buttonAltIdleBg         | `#ffffff`                 |
| buttonAltIdleBorder     | `rgba(0,0,0,0.2)`         |
| buttonAltIdleShadow     | `rgba(0,0,0,0.05)`        |
| buttonAltPassiveBg      | `#757575`                 |
| buttonAltHoverBg        | `#121212`                 |
| buttonAltHoverText      | `#ffffff`                 |
| buttonAltHoverBorder    | `rgba(0,0,0,0.05)`        |
| buttonAltHoverShadow    | `rgba(0,0,0,0.05)`        |
| buttonAltFocusBorder    | `rgba(46, 234, 175, 0.3)` |
| buttonAltFocusShadow    | `rgba(46, 234, 175, 0.3)` |
| buttonSocialIconColor   | `#ffffff`                 |
| buttonSocialIdleText    | `#222222`                 |
| buttonSocialIdleBg      | `#f7f7f7`                 |
| buttonSocialIdleBorder  | `transparent`             |
| buttonSocialIdleShadow  | `transparent`             |
| buttonSocialPassiveBg   | `#757575`                 |
| buttonSocialHoverBg     | `#f7f7f7`                 |
| buttonSocialHoverText   | `#222222`                 |
| buttonSocialHoverBorder | `transparent`             |
| buttonSocialHoverShadow | `transparent`             |
| buttonSocialFocusBorder | `rgba(46, 234, 175, 0.3)` |
| buttonSocialFocusShadow | `rgba(46, 234, 175, 0.3)` |
