# oidc-auth-examples

> Example code snippets for OIDC authentication with ALA systems

- [oidc-auth-examples](#oidc-auth-examples)
- [Introduction](#introduction)
- [Examples](#examples)
  - [Single Page Application](#single-page-application)
    - [PKCE Authentication](#pkce-authentication)
    - [Implicit Authentication](#implicit-authentication)
  - [Expo Application](#expo-application)
    - [PKCE Authentication](#pkce-authentication-1)
    - [Implicit Authentication](#implicit-authentication-1)

# Introduction

This repository provides several examples of how to use the Atlas of Living Australia's new OIDC authentication system.

The **[Atlas of Living Australia API Docs](https://docs.ala.org.au/#authentication)** should also be used as a reference.

# Examples

## Single Page Application

### PKCE Authentication

An example React application which implements SPA PKCE authentication can be found in the [ala-oidc-react](https://github.com/AtlasOfLivingAustralia/ala-oidc-react) repository. This example application uses the [react-oidc-context](https://www.npmjs.com/package/react-oidc-context) package to handle OIDC authentication.

The two key components of this OIDC auth implementation are as follows:

1. The `App.tsx` component includes an authentication provider, like so:

```tsx
import { AuthProvider } from 'react-oidc-context';

<AuthProvider
  client_id='your_client_id'
  authority='your_authority_uri' // I.E. The OIDC discovery endpoint
  redirect_uri='your_redirect_uri'
  onSigninCallback={(user) => console.log('TESTING', user)} // Must be specified
>
  {/* Your application here... */}
</AuthProvider>;
```

2. The authentication page (`src/routes/Auth/index.tsx`) implements the necessary login / logout functionality

```tsx
function Auth(): ReactElement {
  const auth = useAuth();

  // Login / logout handler
  const onClick = useCallback(() => {
    if (auth.isAuthenticated) {
      auth.signoutRedirect();
    } else {
      auth.signinRedirect();
    }
  }, [auth]);

  return (
    <>
      <Button onClick={onClick}>
        {auth.isAuthenticated ? 'Sign Out' : 'Sign In'}
      </Button>
      {auth.isAuthenticated && <div>{JSON.stringify(auth.user, null, 2)}</div>}
    </>
  );
}
```

Please see the [Authentication Code Flow](https://docs.ala.org.au/#authentication-code-flow) section of the ALA API Docs for more information on PKCE authentication.

### Implicit Authentication

**This method of authentication is <u>no longer recommended</u>. Please refer to the PKCE style of authentication above.**

The [ala-web-auth](https://github.com/AtlasOfLivingAustralia/ala-web-auth) repository provides an example of how to use ALA's implicit OIDC authentication for Single Page Applications.

<u>**PLEASE NOTE: This is only proof-of-concept not currently published to NPM, and hence, will need to be built locally before being used.**</u>

The general flow for SPA implicit OIDC Authentication follows as such:

1. The user clicks the login button, and is redirected to the authentication page.
2. The user enters their credentials.
3. Upon successful authentication, the user is redirected back to the SPA with the token provided in the query parameters.

The following code snippit provides an example of this:

```typescript
const onLoginClick = (): void => {
  const client = getClient(
    'oidc-test-client-id',
    ['openid', 'email', 'profile'],
    'test'
  );
  console.log(getAuthUrl(client, 'https://localhost:3000'));
  signInWithRedirect(client, 'http://localhost:3000');
};
```

## Expo Application

### PKCE Authentication

The [biocollect-expo](https://github.com/AtlasOfLivingAustralia/biocollect-expo) repository provides sample code for using OIDC PKCE Authentication, using the [expo-auth-session](https://docs.expo.dev/versions/latest/sdk/auth-session/) package.

Also see the [Authentication Code Flow](https://docs.ala.org.au/#authentication-code-flow) section of the ALA API Documentation.

1. Firstly, the we construct a new auth request object.

```typescript
// Auth Session
import {
  fetchDiscoveryAsync,
  CodeChallengeMethod,
  AuthRequest,
  AccessTokenRequest,
} from 'expo-auth-session';

// Create a deep link for authentication redirects
const redirectUri = Linking.createURL('/auth');

// Construct the OIDC code request
const codeRequest = new AuthRequest({
  clientId: 'client-id-here',
  redirectUri,
  scopes: ['openid', 'email', 'profile'],
  codeChallengeMethod: CodeChallengeMethod.S256,
});
```

**Note:** The supplied `redirectUri` parameter is a _deep link_, meaning that the user will be redirected back within the application upon successfully logging in.

2. Then, we call the `fetchDiscoveryAsync` function to retrieve the ALA's OpenID Connect Discovery metadata and start the authentication flow.

```typescript
// Fetch the discovery metadata
const discovery = await fetchDiscoveryAsync(
  'https://auth-test.ala.org.au/cas/oidc'
);

// Start the authentication flow
const result = await codeRequest.promptAsync(discovery);
```

Upon successful authentication, we're provided with an [AuthSessionResult](https://docs.expo.dev/versions/latest/sdk/auth-session/#authsessionresult) object.

We use this code to finally construct & perform a token request, which retireves the user's access token and ID token.

```typescript
// Construct the OIDC access token request
const tokenRequest = new AccessTokenRequest({
  clientId: codeRequest.clientId,
  scopes: codeRequest.scopes,
  code: result.params.code,
  redirectUri,
  extraParams: {
    code_verifier: codeRequest.codeVerifier,
  },
});

const accessToken = await tokenRequest.performAsync(discovery);
```

The `performAsync()` method returns a [TokenResponse](https://docs.expo.dev/versions/latest/sdk/auth-session/#tokenresponseconfig) object, which can be used for subsequent authentication requests.

### Implicit Authentication

**This method of authentication is <u>no longer recommended</u>. Please refer to the PKCE style of authentication below.**

The general flow for Expo implicit OIDC Authentication follows as such:

1. The user clicks the sign-in button, which opens a web browser popup on the ALA authentication page.
2. The user enters their credentials.
3. Upon successful authentication, the user redirected back into the application via a universial (iOS) / deep (Android) link, proivded with the credentials.

The following is a simple authentication code snippit from the repository:

```typescript
import * as Linking from 'expo-linking';
import {
  openAuthSessionAsync,
  WebBrowserRedirectResult,
} from 'expo-web-browser';

// Authentication handler
const handleAuth = async (): Promise<void> => {
  const client_id = 'client-id-here';
  const redirect_uri = Linking.createURL('/auth');

  // Start the authentication session
  const result = await openAuthSessionAsync(
    `https://auth-test.ala.org.au/cas/oidc/oidcAuthorize?client_id=${client_id}&redirect_uri=${redirect_uri}&response_type=token&scope=openid%20email%20profile`,
    redirect_uri
  );

  // Navigate to the home screen upon successful authentication
  if (result.type === 'success') props.navigation.navigate('Home');
};
```
