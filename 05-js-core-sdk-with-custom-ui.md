<!-- TODO:
      - Add link to the example repo
      - Add section to handle oauth
      - Add link to typedocs
-->

# Using the Javascript Core SDK with custom UI

If you would like to use a custom UI with scute, you can do so using the `@scute/core` package. This package would expose the `scuteClient` which you can use to easily hook your UI to scute API. For this example we will be using react as our client side framework and use the `@scute/core` package to implement the authentication flow with a custom flow.

Head over to the [example project repo](#) to clone and run this example project and check out the [type docs](#) for more `scuteClient` methods.

## Initialize the `scuteClient`

Get your scute credentials [as described here](./01-getting-credentials.md) and add `@scute/core` to your project

Initialize the scute client:

```js
// scute.js
import { createClient } from "@scute/core";

export const scuteClient = createClient({
  appId: "your_app_id",
  baseUrl: "your_base_url",
});
```

## Sign In or Sign Up

### Build the form

Build an HTML form with an email field for the identifier and a submit button.

```jsx
import { scuteClient } from "./scute.js";
// ... Redacted

const [email, setEmail] = useState("");

<div>
  <div>
    <input
      type="email"
      value={email}
      placeholder="Email"
      onChange={(e) => setEmail(e.target.value)}
    />
  </div>
  <br />
  <div>
    <button>Sign In or Sign Up</button>
  </div>
</div>;
```

### Add meta fields

If you have set up meta fields [at control.scute.io](https://control.scute.io), you can get those using the `scuteClient.getAppData();` method of the scute client inside a `useEffect` hook. Like most of the client methods, this will return an object that has 2 properties: `error` and `data`. If the request is successful you will get an object for `data` which contains the meta field information as an array under `user_meta_data_schema`. You can loop over this data and add your meta fields to the form as well.

```jsx
const [metaFields, setMetaFields] = useState({});

useEffect(() => {
  const getAppData = async () => {
    const { data, error } = await scuteClient.getAppData();

    const metaFormData = data.user_meta_data_schema.reduce(
      (acc, field) => ({
        ...acc,
        [field.field_name]: "",
      }),
      {}
    );
    setMetaFields(metaFormData);
  };

  getAppData();
}, [])

// ...Redacted

<>
  {Object.keys(metaFields).map((fieldName) => (
    <div key={fieldName}>
      <input
        value={metaFields[fieldName]}
        type="text"
        placeholder={fieldName}
        onChange={(e) =>
          setMetaFields({ ...metaFields, [fieldName]: e.target.value })
        }
      />
    </div>
  ))}
</>

```

### Initiate the Sign In or Sign Up flow

You can use the `scuteClient.signInOrUp` method to initiate the authentication flow. The `signInOrUp` method is a wrapper method that takes care of a few things:

- It will create a user with the `email` passed to it if the user doesn't exist and dispatch a magic link to their inbox.
- If the user already exists but does not have a passkey stored, it will start the magic link authentication flow, dispatching a magic link to the entered email.
- If the user already exists and have a passkey stored, it will start device verification flow prompting the user for their passkey via their biometrics or other authentication devices like a usb key.

Update your button to use `scuteClient.signInOrUp`, pass in the email from the form and the meta fields if you set them up. The method will return an object with `data` and `error` as usual.

If both `data` and `error` are `undefined`, It means that the user is authenticated using a passkey, you are good to let them see the protected information. Otherwise you will get a `magic_link` object inside `data`, that contains your magic link `id`.

```js
// ... Redacted

const [magicLink, setMagicLink] = useState(null);

if (magicLink) {
  return <p>Please Check Your Email</p>;
}

// ... Redacted

<button
  onClick={async () => {
    const { data, error } = await scuteClient.signInOrUp(email, {
      // if you set them up pass the meta data
      userMeta: metaFields,
    });

    if (error) {
      console.error(error);
      return;
    }

    if (!data) {
      // passkey authentication successful, user is signed in
      // take user to their profile
      router.push("/profile");
    } else {
      // magic link
      setMagicLink(true);
    }
  }}
>
  Sign In or Sign Up
</button>;
```

The user will get an email with a magic link that looks like `<YOUR_BASE_URL>/?sct_magic=<MAGIC_LINK_TOKEN>`.

## Handling the magic link

### Verifying the magic link token and signing the user in

Once the user clicks the magic link, they will be redirected back to your application with a magic link token in the url. Make sure to catch this in your code and verify the token. Once the token is verified you will have your session information. Using the `scuteClient.signInWithTokenPayload` you can authenticate the user and let them in:

```jsx
useEffect(() => {
  const verifyMagicLink = async () => {
    const magicLinkToken = scuteClient.getMagicLinkToken();
    const { data: payloads, error } = await scuteClient.verifyMagicLinkToken(
      magicLinkToken
    );

    if (error) {
      return console.error(error);
    }

    // At this point you have your session. Authenticate the user and let them in.
    const { data, error } = await scuteClient.signInWithTokenPayload(
      payloads.authPayload
    );
    router.push("/profile");
  };

  verifyMagicLink();
}, []);
```

## Passkeys

### Registering a device with a passkey

To register a device with a passkey, we need to modify the above hook using the `scuteClient.addDevice` method:

```jsx
useEffect(() => {
  const verifyMagicLink = async () => {
    const magicLinkToken = scuteClient.getMagicLinkToken();
    const { data: payloads, error } = await scuteClient.verifyMagicLinkToken(
      magicLinkToken
    );

    if (error) {
      return console.error(error);
    }

    const { data, error } = await scuteClient.signInWithTokenPayload(
      payloads.authPayload
    );
    // At this point you are signed in, prompt user for device registration
    const { data, error } = await scuteClient.addDevice();
    // Device is registered let the user in
    router.push("/profile");
  };

  verifyMagicLink();
}, []);
```

## Signing out the user

To sign the user out use the `scuteClient.signOut` method:

```jsx
<button
  onclick={() => {
    scuteClient.signOut();
    router.push("/");
  }}
>
  Sign Out
</button>
```
