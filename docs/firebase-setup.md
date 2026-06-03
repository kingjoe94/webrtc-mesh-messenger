# Firebase Signaling Setup

This app can use Firebase Firestore as a short-lived signaling provider for already paired friends.

Message bodies must not be written to Firestore. Firestore is only for reconnect signaling data such as:

- reconnect request
- offer
- answer
- ICE candidate

## Firebase Console Steps

1. Create a Firebase project.
2. Add a Web app.
3. Copy the Web app config object.
4. Enable Cloud Firestore.
5. Start in production mode, then apply rules before testing from the public URL.
6. Replace `FIREBASE_CONFIG = null` in `index.html` with the copied config object.

Example:

```js
const FIREBASE_CONFIG = {
  apiKey: "...",
  authDomain: "...firebaseapp.com",
  projectId: "...",
  storageBucket: "...firebasestorage.app",
  messagingSenderId: "...",
  appId: "..."
};
```

Firebase Web config is not a private secret. Firestore Rules are the important boundary.

## Test Rules

These rules are intentionally narrow but still suitable only for MVP testing.

```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /devices/{deviceId}/signals/{signalId} {
      allow read: if true;
      allow create: if
        request.resource.data.keys().hasOnly([
          'kind', 'from', 'to', 'createdAt', 'expiresAt',
          'mode', 'sdp', 'candidate'
        ]) &&
        request.resource.data.to == deviceId &&
        request.resource.data.from is string &&
        request.resource.data.to is string &&
        request.resource.data.kind is string &&
        request.resource.data.expiresAt is timestamp &&
        request.resource.data.expiresAt > request.time &&
        request.resource.data.expiresAt < request.time + duration.value(10, 'm') &&
        request.resource.data.from.size() <= 64 &&
        request.resource.data.to.size() <= 64 &&
        request.resource.data.kind in ['reconnect-request', 'offer', 'answer', 'ice'];
      allow update, delete: if false;
    }
  }
}
```

## Cleanup

Configure Firestore TTL on `expiresAt` for:

```txt
devices/{deviceId}/signals/{signalId}
```

TTL cleanup is not immediate, so the app must still ignore expired signaling data.

## Current Limit

This is only the Firebase configuration and provider shell. The next step is to route saved-friend reconnect offer, answer, and ICE exchange through the provider.
