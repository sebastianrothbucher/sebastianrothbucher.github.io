---
layout: post
title: DIY-ing a (Vue) app with AppSync for GraphQL and Cognito for (O)Auth
date: 2024-03-07 11:00:00 +0000
categories: aws appsync cognito cloudnative oauth2 bootstrap web javascript typescript vue
---

Cloud-native is quite powerful as there's hardened components doing much of the work. With this post, I'm bringing together some great services (on AWS) and some great libraries from the open source space to cover the bases needed for a real-live web app. 

Even though it's a great service, I'll *not* use Amplify (which btw was simplified a little more - see [this re:invent talk](https://www.youtube.com/watch?v=UfYWGYbmV3s)). I'll really bring the pieces together myself - which conveniently also allows for a look under the hood. 

## Libraries used

I'll follow the JAM-stack pattern, i.e. UI and backend are separate, tokens are used for authentication. I'll be using these libraries.

- of course, [Vue](https://vuejs.org/) (my fave, even though Angular and React are great as well). I'm using Vue3 with [Pinia](https://pinia.vuejs.org/) for state, [Vite](https://vitejs.dev/) for build and [Vitest](https://vitest.dev/) for test. 
- [Typescript](https://www.typescriptlang.org/)
- [@badgateway/oauth2-client](https://www.npmjs.com/package/@badgateway/oauth2-client) - really great and compact impl of OAuth2
- [@urql/core](https://www.npmjs.com/package/@urql/core) - the URQL library for accessing GraphQL. There is a ready integration with Vue - yet I've decided to only use the base lib and do plumbing myself
- [@graphql-codegen/cli](https://www.npmjs.com/package/@graphql-codegen/cli) and [@graphql-codegen/typescript](https://www.npmjs.com/package/@graphql-codegen/typescript) - optional (but helpful): allows for typing via `import type`

## Services used (on AWS)

- [AppSync](https://docs.aws.amazon.com/appsync/) - implementation of GraphQL. Requests are mapped to either Lambdas or direct database access, etc. I won't go into the details of AppSync, but there's a lot of great material out there
- [Cognito](https://docs.aws.amazon.com/cognito/) - an OAuth implementation managing users login (and optionally signup). Provides a login page ready to use - and leaves it to AWS to rack their brains on how to keep things secure. 

## Getting authentication in place

First off, let's call an `ensureLogin` method to make sure we have a user logged in already. I put it inside a Pinia store, but that's totally optional: 

```javascript
const loginInfo = { currentToken: null, currentRefreshToken: null, currentUserName: null};
async function ensureLogin(): Promise<boolean> {
  await initPromise; // wait till auth is initialized & make sure we got current info - see below
  if (!loginInfo.currentToken) { // redirect to log us in
    redirectToLogin();
    return false;
  }
  return true;
}
```

As you probably know, the way OAuth roughly works browser-side is by redirecting to a login page outside our app, have the user login there, redirect back with a temporary code, our app takes the code to get the "actual" token (aka *access token* - short-lived) along with a possibility to get a new access token when expired (aka *refresh-token*). There is also an ID token which has more info on the user - which we don't need here. 

So, as we don't have a token initially, let's take on the step of going to the login page. Btw I also did this with a lot of help by the great docs of [@badgateway/oauth2-client](https://www.npmjs.com/package/@badgateway/oauth2-client)

```javascript
import { OAuth2Client, OAuth2Fetch, generateCodeVerifier } from '@badgateway/oauth2-client';
const client = new OAuth2Client({
  server: 'https://<your user pool>.auth.eu-central-1.amazoncognito.com/', // slash counts! (your region instead of eu-central-1)
  clientId: '<your client ID>',
  authorizationEndpoint: 'login', // Cognito
  tokenEndpoint: 'token',
});
const redirectUri = 'http://localhost:8080/'; // (or the app, later - can of course use environments, just KISS here)
async function redirectToLogin() {
  const codeVerifier = await generateCodeVerifier();
  localStorage.setItem("pkce_code_verifier", codeVerifier);
  document.location = await client.authorizationCode.getAuthorizeUri({
    redirectUri,
    codeVerifier,
    scope: ['openid', 'email', 'aws.cognito.signin.user.admin'],
  });
}
```

Some comments here:

- a verifier code (a value we pass to Cognito and get back) is not strictly necessary, but helps keep things as secure as we can (and we sure want that)
- we need the `aws.cognito.signin.user.admin` scope to later pull the username

Now the user will be presented with a login page by Cognito to log in. Upon successful login, we go back to our redirect URL along with a code from Cognito and our verifier code. Here's the method to handle that

```javascript
async function handleRedirectBack() {
  if (!location.search?.includes('code')) {
    return;
  }
  const codeVerifier = localStorage.getItem("pkce_code_verifier");
  let oauth2Token, oauthUserInfo;
  try {
    oauth2Token = await client.authorizationCode.getTokenFromCodeRedirect(
      document.location as any,
      {
        redirectUri,
        codeVerifier,
      }
    );
    oauthUserInfo = await fetch(client.settings.server + 'oauth2/userInfo', {headers: {'Authorization': 'Bearer ' + oauth2Token.accessToken}}).then(res => res.json());
  } catch (error) {
      alert("Error returned from authorization server: " + error);
      return;
  }
  localStorage.removeItem("pkce_code_verifier");
  window.history.replaceState({}, null, "/");
  loginInfo.currentToken = oauth2Token.accessToken;
  loginInfo.currentRefreshToken = oauth2Token.refreshToken;
  loginInfo.currentUserName = oauthUserInfo?.email;
  console.log(loginInfo.currentUserName);
}
initPromises.push(handleRedirectBack());
```

Some comments here as well:

- as we're re-entering our app now, we need to react immediately upon app load
- to make sure our own logic runs after, we collect the promise(s) upon load (we'll bring it all together below)
- apart from that, it's pretty straightforward: we get some params from Cognito in the URL, parse them, save our tokens
- as we want more info (i.e. the e-mail) we also call Cognito's user info endpoint (little sad the e-mail is not just included)

Now, we're almost there. A great feature of [@badgateway/oauth2-client](https://www.npmjs.com/package/@badgateway/oauth2-client) is providing a modified version of `fetch` that injects the token and also takes care of refreshing (if needed). Here's how you get the modified `fetch`:

```javascript
const fetchWrapper = new OAuth2Fetch({
  client,
  getNewToken: async () => {
    const newOauth2Token = await client.refreshToken({
      refreshToken: loginInfo.currentRefreshToken,
    } as any);
    loginInfo.currentToken = newOauth2Token.accessToken;
    loginInfo.currentRefreshToken = newOauth2Token.refreshToken;
    return newOauth2Token;
  },
  onError: (error) => alert("Error returned from authorization server: " + error),
});
```

The `fetchWrapper.fetch` method can be called just like a regular `fetch` now.

Finally, when it comes to init: 

```javascript
const initPromise = Promise.all(initPromises);
```

This allows us to wait until we ran through the init steps before checking for login (again) and then poss. making calls to the API.

With that, we have everything in place to make calls to APIs secured by our Cognito user pool. If we were using REST (e.g. with [API Gateway](https://docs.aws.amazon.com/apigateway/)), we'd be done already. As we want to get GraphQL in, let's create an [@urql/core](https://www.npmjs.com/package/@urql/core) client and get GraphQL rolling. 

## Making a GraphQL call

You can create a basic client like this (using the `fetchWrapper` from above): 

```javascript
import { Client as GraphqlClient, fetchExchange } from '@urql/core'
const graphqlClient = new GraphqlClient({
  exchanges: [fetchExchange],
  url: 'https://<our API>.appsync-api.eu-central-1.amazonaws.com/graphql', // (your region instead of eu-central-1)
  fetch: fetchWrapper.fetch.bind(fetchWrapper),
});
```

Which is pretty straightforward already - now, we can do a call like this (assuming we have a query `owned` returning `Items` with `id` and `title` in each item): 

```javascript
import { gql } from '@urql/core'
const ownedQuery = gql`query {
  owned {
    Items {
        id
        title
    }      
  }
}`
const ownedInfo = await graphqlClient.query(ownedQuery, null);
if (ownedInfo.error) {
  console.error(ownedInfo.error);
  return;
}
ownedInfo.data?.owned?.Items // do sth with it
```

The backtick syntax looks a little strange - in the end, it's a function call & nothing more. There is btw no pre-processing or sth needed, but let's stick to the syntax that's customary for GraphQL even though we don't have to.

## Using types

As said, types are optional. One can use [@graphql-codegen/cli](https://www.npmjs.com/package/@graphql-codegen/cli) along with [@graphql-codegen/typescript](https://www.npmjs.com/package/@graphql-codegen/typescript) to basically create interfaces and enums and have Typescript support development. 

There **is one big caveat, though:** so far, *Vite* can't handle the output of *@graphql-codegen* beyond type declarations - the build in *Vite* will simply fail. However, it *works great* with `import type`, i.e. instead of 

```javascript
import  { TypeFromGqlSchema } from '@/types/my-graphql-types'
```

use

```javascript
import type /* !!!!! */ { TypeFromGqlSchema } from '@/types/my-graphql-types'
```

and y're good. 

## Wrap-up

OAuth2 along with a managed service (there are others besides Cognito) are quite handy. Guess at some point we need to tweak things back to old-school sessions - namely when quantum computing makes JWT as such too insecure. In any case, security is tough and getting tougher by the day. A managed service as shown above offloads most of it to a team at AWS that has more bandwith to deal with it.
