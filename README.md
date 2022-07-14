# oidc-auth-examples

> Example code snippets for OIDC authentication with ALA systems

- [oidc-auth-examples](#oidc-auth-examples)
  - [Introduction](#introduction)
  - [Examples](#examples)
    - [Single Page Application](#single-page-application)
      - [Implicit Authentication](#implicit-authentication)
      - [PKCE Authentication](#pkce-authentication)
      - [React Wrapper](#react-wrapper)
    - [Expo Application](#expo-application)
      - [Implicit Authentication](#implicit-authentication-1)
      - [PKCE Authentication](#pkce-authentication-1)

## Introduction

This repository provides several examples of how to use the Atlas of Living Australia's new OIDC authentication system.

The **[Atlas of Living Australia API Docs](https://docs.ala.org.au/#authentication)** should also be used as a reference.

## Examples

### Single Page Application

#### Implicit Authentication

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
    "oidc-test-client-id",
    ["openid", "email", "profile", "users:read"],
    "test"
  );
  console.log(getAuthUrl(client, "https://localhost:3000"));
  signInWithRedirect(client, "http://localhost:3000");
};
```

When the page loads, the token is retireved using the `getRedirectResult()` function and stored using [Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage).

#### PKCE Authentication

The [ala-web-auth](https://github.com/AtlasOfLivingAustralia/ala-web-auth) does not support PKCE authentication (yet) as it is simply a proof of concept, however, the PKCE authentication under the 'Expo' section below explains an approach to implementing it when authenitcating with ALA.

Please also see the [Authentication Code Flow](https://docs.ala.org.au/#authentication-code-flow) section of the ALA API Docs for more information on PKCE authentication.

#### React Wrapper

Libraries such as [react-oidc-context](https://github.com/authts/react-oidc-context) can be used to provide OIDC functionality when using React.

For more information, please view the repository README.

### Expo Application

#### Implicit Authentication

The [biocollect-expo](https://github.com/AtlasOfLivingAustralia/biocollect-expo) repository provides sample code for using `implicit` OIDC Authentication, using the [expo-web-browser](https://docs.expo.dev/versions/latest/sdk/webbrowser/) and [expo-linking](https://docs.expo.dev/versions/latest/sdk/linking/) packages.

The general flow for Expo implicit OIDC Authentication follows as such:

1. The user clicks the sign-in button, which opens a web browser popup on the ALA authentication page.
2. The user enters their credentials.
3. Upon successful authentication, the user redirected back into the application via a universial (iOS) / deep (Android) link, proivded with the credentials.

The following is a simple authentication code snippit from the repository:

```typescript
import * as Linking from "expo-linking";
import {
  openAuthSessionAsync,
  WebBrowserRedirectResult,
} from "expo-web-browser";

// Authentication handler
const handleAuth = async (): Promise<void> => {
  const client_id = "client-id-here";
  const redirect_uri = Linking.createURL("/auth");

  // Start the authentication session
  const result = await openAuthSessionAsync(
    `https://auth-test.ala.org.au/cas/oidc/oidcAuthorize?client_id=${client_id}&redirect_uri=${redirect_uri}&response_type=token&scope=openid%20email%20profile`,
    redirect_uri
  );

  // Navigate to the home screen upon successful authentication
  if (result.type === "success") props.navigation.navigate("Home");
};
```

#### PKCE Authentication

The above example can be modified to support PKCE authentication, with the following changes to the query parameters:

```typescript
{
	response_type: 'code',
	code_challenge_method: 'S256', // SHA-256 Encoding
	code_challende: 'generated-code'
}
```

- More information about PKCE is available [here](https://oauth.net/2/pkce/).
- Also see the [Authentication Code Flow](https://docs.ala.org.au/#authentication-code-flow) section of the ALA API Documentation.
