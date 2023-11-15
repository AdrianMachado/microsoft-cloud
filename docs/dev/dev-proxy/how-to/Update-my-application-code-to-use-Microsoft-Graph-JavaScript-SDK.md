---
title: Update my application code to use Microsoft Graph JavaScript SDK
description: How to update your application code to use the Microsoft Graph JavaScript SDK
author: garrytrinder
ms.author: garrytrinder
ms.date: 11/03/2023
ms.topic: conceptual
ms.service: microsoft-cloud-for-developers

categories:
  - developer-tools
products:
  - microsoft-365
  - microsoft-graph
  - sharepoint-online
  - m365
ms.custom:
  - fcp
  - team=cloud_advocates
---

# Update my application code to use Microsoft Graph JavaScript SDK

The Microsoft Graph JavaScript SDK comes with features that simplifies your code and let you focus on building your app. 

Using the SDK makes it easier for you to:

- Handle API errors for when things don't work as expected, such as when services throttle under heavy load
- Perform complex API operations like batch requests
- Handle binary responses, such as obtaining the photo of a user

## Migrate from fetch to the Graph JavaScript SDK

If you use the fetch API to call APIs in your JavaScript app, you might have code similar to the following:

```javascript
const msalClient = new msal.PublicClientApplication({
  auth: {
    clientId: appId
  }
});

function getAccessToken(msalClient) {
  const accounts = msalClient.getAllAccounts();

  if (accounts.length > 0) {
    const accessTokenRequest = {
      scopes: [
        'https://graph.microsoft.com/User.Read'
      ],
      account: accounts[0]
    };

    return msalClient.acquireTokenSilent(accessTokenRequest)
      .then(response => response.accessToken)
      .catch(error => {
        console.log(error);
        console.log("silent token acquisition fails. acquiring token using redirect");
        if (error instanceof msal.InteractionRequiredAuthError) {
          return msalClient.acquireTokenRedirect(accessTokenRequest);
        }
      });
  }
  else {
    return Promise.reject('Sign in');
  }
}

msalClient
  .loginPopup()
  .then(response => getAccessToken(msalClient))
  .then(accessToken => fetch('https://graph.microsoft.com/v1.0/me', {
    method: 'GET',
    headers: {
      authorization: `Bearer ${accessToken}`
    }
  }))
  .then(response => response.json())
  .then(json => {
    // do something here
  });
```

To use the Graph JavaScript SDK, you'd change the code to:

```javascript
const msalClient = new msal.PublicClientApplication({
  auth: {
    clientId: appId
  }
});

function getGraphClient(account) {
  const authProvider = new MSGraphAuthCodeMSALBrowserAuthProvider.AuthCodeMSALBrowserAuthenticationProvider(msalClient, {
    account,
    scopes: [
      'https://graph.microsoft.com/User.Read'
    ],
    interactionType: msal.InteractionType.Popup,
  });

  return MicrosoftGraph.Client.initWithMiddleware({ authProvider });
}

msalClient
  .loginPopup()
  .then(response => {
    const accounts = msalClient.getAllAccounts();

    if (accounts.length > 0) {
      const graphClient = getGraphClient(accounts[0]);
      return graphClient.api('/me').get();
    }
    else {
      return Promise.reject('Sign in');
    }
  })
  .then(json => {
    // do something here
  });
```

## Handle API errors

One of the most common API errors that applications using Microsoft Graph experience when used at scale is throttling. It occurs, when the server is under heavy load. Throttling decreases the load on the server to keep the service up.

Since throttling rarely occurs on developer tenants, often developers call the API without properly handling errors:

```javascript
fetch('https://graph.microsoft.com/v1.0/me', {
    method: 'GET',
    headers: {
      authorization: `Bearer ${accessToken}`
    }
  })
  .then(response => response.json())
  .then(json => {
    // do something here
  });
```

The proper way to handle throttling errors with fetch API would be to extend the call to watch out for 429 throttling errors and wait before calling the API again for the number of seconds designated in the `retry-after` response header. Updated code would look as follows:

```javascript
function sleep(milliseconds) {
  return new Promise((resolve) => setTimeout(resolve, milliseconds));
}

async function fetchAndRetryIfNecessary(callAPIFn) {
  const response = await callAPIFn();

  if (response.status === 429) {
    const retryAfter = response.headers.get('retry-after');
    await sleep(retryAfter * 1000);
    return fetchAndRetryIfNecessary(callAPIFn);
  }

  return response;
}

const response = await fetchAndRetryIfNecessary(async () => (
  await fetch('https://graph.microsoft.com/v1.0/me', {
    method: 'GET',
    headers: {
      authorization: `Bearer ${accessToken}`
    }
  })
));
const json = await response.json();
// do something here
```

An easier way to handle throttling, and other errors, is to use the Graph JavaScript SDK, which handles errors for you.

```javascript
const json = await graphClient.api('/me').get();
// do something here
```