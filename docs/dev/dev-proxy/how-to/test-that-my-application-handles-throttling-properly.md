---
title: Test that my application handles throttling properly
description: How to test that your application handles throttling properly
author: garrytrinder
ms.author: garrytrinder
ms.date: 11/03/2023
---

# Test that my application handles throttling properly

Typically, testing throttling is hard because it occurs rarely, when Microsoft 365 servers are under heavy load. Using the Dev Proxy, you can mimic throttling responses, and check if your application handles it correctly.

To test your application, start the Dev Proxy, configuring it to respond specifically with throttling responses, intercepting a high percentage of requests:

```sh
devproxy --allowed-errors 429 --failure-rate 90 --no-mocks
```

Next, run your application, and verify that it doesn't break when it gets 429 responses but waits for the amount of time specified on the response.

When testing your app, pay attention to the Dev Proxy output.

If your application backs-off when throttled, but doesn't wait for the amount of time specified on the requests, you see a message similar to `Calling https://graph.microsoft.com/v1.0/endpoint again before waiting for the Retry-After period. Request will be throttled`.

This message indicates that your application isn't handling throttling correctly and will stay throttled.