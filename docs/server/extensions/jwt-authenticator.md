# Extension JWT Authentication

Our JWT extension provides an easy way to authenticate to Hocuspocus using json web tokens.

## Installation

:::warning Pro Feature
To get started, you need a Tiptap Pro account ([sign up / login here](https://tiptap.dev/pro)).
:::

Install the JWT Authenticator extension like this:

```bash
npm install @hocuspocus-pro/extension-jwt-authenticator
```

## Configuration

**isActive** can be used to enable/disable the extension during runtime. You can pass a boolean or an async callback that resolves to a boolean.

**secretOrPublicKey** this is the secret (used to encode the JWT, or a public key that can be used to verify the signature. This is passed directly to jsonwebtoken.verify, additional docs are available [here](https://github.com/auth0/node-jsonwebtoken#jwtverifytoken-secretorpublickey-options-callback).

## Usage

```typescript
const server = Server.configure({
  extensions: [
    new JWTAuthenticator({
      isActive: true,
      secretOrPublicKey: 'my-custom-jwt-secret',
    }),
  ],
});

server.listen();
```

## Context

Once loaded, the authentication will add the following types to the `context`:

```typescript
type JWTContext = {
    jwtAuthenticator: {
        isAuthenticated: boolean // whether or not the JWT has been used for authentication
        allowedDocumentNames?:string[], // array of document names that are permitted by the JWT
        context?: Record<string, unknown> // decoded part of the JWT (return value of `jsonwebtoken.verify`)
        userId?: unknown // `sub` field of the JWT
        token?: string // the JWT passed by the user
    }
}
```

## Generating the JWT

The JWT **must** be generated on the server side, as your `secret` **must not** leave your server (i.e. don't even try to generate the JWT on the frontend).
You can use the following snippet on a NodeJS server and build an API around it.

```typescript
import jsonwebtoken from 'jsonwebtoken'

const jwt = jsonwebtoken.sign({ /* object to be encoded in the JWT */ }, 'your_secret')
// this JWT should be sent in the `token` field of the provider. Never expose 'your_secret' to a frontend!
```

A full server / API example is available [here](https://github.com/ueberdosis/tiptap-collab-replit/blob/main/src/server-collab.ts).
Make sure to put the `secret` inside the server environment variable (or just make it a constant in the server file, don't transfer it from the client).
You probably want to create an API call like `GET /getCollabToken` which will generate the JWT based on the server secret and the list of documents that the user is allowed to access.

### How to limit access to specific documents

Documents can only be accessed by knowing the exact document name, as there is no way to get a list of documents from TiptapCollab.
Thus, it's a good practice to name them like `userUuid/documentUuid` (i.e. `1500c624-8f9f-496a-b196-5e5dd8ec3c25/7865975c-38d0-4bb5-846b-df909cdc66d3`), which
already makes it impossible to open random documents by guessing the name.

If you want to further limit which documents can be accessed using which JWT, you can encode the `allowedDocumentNames` property in the JWT, as in the following
example. The created JWT will only allow access to the document(s) specified.

```typescript
import jsonwebtoken from 'jsonwebtoken'

const jwt = jsonwebtoken.sign({
  allowedDocumentNames: [
    '1500c624-8f9f-496a-b196-5e5dd8ec3c25/7865975c-38d0-4bb5-846b-df909cdc66d3', // userUuid/documentUuid
    '1500c624-8f9f-496a-b196-5e5dd8ec3c25/*' // userUuid/*
  ]
}, 'your_secret')
// this JWT should be sent in the `token` field of the provider. Never expose 'your_secret' to a frontend!
```


## Tutorial

Click [here](https://tiptap.dev/tutorials/jwt-authentication) to learn more
