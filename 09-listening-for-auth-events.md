# Listening for Auth Events

Scute provides a way to listen for auth events. This is useful if you want to know when a user has signed up, signed in, or signed out.

To listen for auth events, you can use the `onAuthStateChange` method of the `ScuteClient` instance. This method takes a callback function that will be called with the auth event object.

The `onAuthStateChange` method returns an unsubscribe function that you can use to stop listening for auth events.

```javascript
import {
  AUTH_CHANGE_EVENTS,
  ...
} from "@scute/core";

...

const [authState, setAuthState] = useState();

useEffect(() => {
  const unsubscribe = scuteClient.onAuthStateChange(
    (event, session, user) => {
      console.log(event, session, user);
    }
  );

  return () => unsubscribe();
}, [scuteClient]);

...
```

The `onAuthStateChange` callback function will be called with the following arguments:

1. `event`: Name of the auth event.
2. `session`: The session object.
3. `user`: The user object.

The `event` argument will be one of the following values:

- `AUTH_CHANGE_EVENTS.SIGNED_IN`: The user has signed in.
- `AUTH_CHANGE_EVENTS.SIGNED_OUT`: The user has signed out.
- `AUTH_CHANGE_EVENTS.INITIAL_SESSION`: Fired when the initial session (or SDK) is loaded.
- `AUTH_CHANGE_EVENTS.SESSION_REFETCHED`: Fired when the session is refetched.
- `AUTH_CHANGE_EVENTS.SESSION_EXPIRED`: Fired when the session has expired.
- `AUTH_CHANGE_EVENTS.TOKEN_REFRESHED`: Fired when the token is refreshed.
- `AUTH_CHANGE_EVENTS.MAGIC_PENDING`: Fired when magick link token verification is pending.
- `AUTH_CHANGE_EVENTS.MAGIC_VERIFIED`: Fired when magick link token verification is successful.
- `AUTH_CHANGE_EVENTS.WEBAUTHN_REGISTER_STARTED`: Fired when WebAuthn device registration starts.
- `AUTH_CHANGE_EVENTS.WEBAUTHN_REGISTER_SUCCESS`: Fired when WebAuthn device registration completes successfully.
- `AUTH_CHANGE_EVENTS.WEBAUTHN_REGISTER_ERROR`: Fired when WebAuthn device registration encounters an error.
- `AUTH_CHANGE_EVENTS.WEBAUTHN_VERIFY_START`: Fired when WebAuthn device verification starts for a previously registered device.
- `AUTH_CHANGE_EVENTS.WEBAUTHN_VERIFY_SUCCESS`: Fired when WebAuthn sign-in completes successfully.
