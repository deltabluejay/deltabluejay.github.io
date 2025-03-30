---
title: Breaking Bank
description: >-
  JKU header injection
categories: [CTF Writeup]
tags: [web, HTB]
pin: false
media_subpath: '/assets/img/posts/Breaking_Bank'
---

## Description
> In the sprawling digital expanse of the Frontier Cluster, the Frontier Board seeks to cement its dominance by targeting the cornerstone of interstellar commerce: Cluster Credit, a decentralized cryptocurrency that keeps the economy alive. With whispers of a sinister 51% attack in motion, the Board aims to gain majority control of the Cluster Credit blockchain, rewriting transaction history and collapsing the fragile economy of the outer systems. Can you hack into the platform and drain the assets for the financial controller?
{: .prompt-info }

## Writeup
This challenge provided a website and its source code. Opening up the challenge reveals a login/registration page:

![login](login.png)
_Login_

After creating an account with some email/password (`a@a` and `a`) and logging in, I was greeted with a cryptocurrency "exchange" website:

![homepage](homepage.png)
_Homepage_

Turns out that the user account we're logged in with is broke, though.

![portfolio](portfolio.png)
_Portfolio_

The website also lets you send friend requests and make transactions. Interestingly, the website notes that transactions require you to be friends and that global transactions are disabled for "security reasons", which seems like a good place to start.

![transactions](transactions.png)
_Transactions_

Looking at the provided source code, in `challenge/server/routes/dashboard.js` we find that the objective of the challenge is to empty the financial controller account's assets:

```js
import { checkFinancialControllerDrained } from '../services/flagService.js';

export default async function dashboardRouter(fastify) {
    fastify.get('/', async (req, reply) => {
        if (!req.user) {
            reply.status(401).send({ error: 'Unauthorized: User not authenticated' });
            return;
        }

        const { email } = req.user;

        if (!email) {
            reply.status(400).send({ error: 'Email not found in token' });
            return;
        }

        const { drained, flag } = await checkFinancialControllerDrained();

        if (drained) {
            reply.send({ message: 'Welcome to the Dashboard!', flag });
            return;
        }

        reply.send({ message: 'Welcome to the Dashboard!' });
    });
}
```

How can we accomplish this then? We'll likely need to utilize the transaction feature to transfer money out of the controller's account. However, to do this, we'll need them to be friends with our user and be able to make a transaction from their account, both of which will require authenticating as the financial controller. The password to the account is randomized on start, so we'll need to find some sort of authentication bypass.

After digging through the code some more to understand the layout, I eventually noticed three "TODO" comments scattered throughout the files- all three of these comments ended up being pertinent to solving the challenge.

Starting with the first vulnerability, the TODO comment points out that the `verifyToken` function may not implement a strong enough check:

`verifyToken` in `challenge/server/services/jwksService.js`:
```js
export const verifyToken = async (token) => {
    try {
        const decodedHeader = jwt.decode(token, { complete: true });

        if (!decodedHeader || !decodedHeader.header) {
            throw new Error('Invalid token: Missing header');
        }

        const { kid, jku } = decodedHeader.header;

        if (!jku) {
            throw new Error('Invalid token: Missing header jku');
        }

        // TODO: is this secure enough?
        if (!jku.startsWith('http://127.0.0.1:1337/')) {
            throw new Error('Invalid token: jku claim does not start with http://127.0.0.1:1337/');
        }

        if (!kid) {
            throw new Error('Invalid token: Missing header kid');
        }

        if (kid !== KEY_ID) {
            return new Error('Invalid token: kid does not match the expected key ID');
        }

        let jwks;
        try {
            const response = await axios.get(jku);
            if (response.status !== 200) {
                throw new Error(`Failed to fetch JWKS: HTTP ${response.status}`);
            }
            jwks = response.data;
        } catch (error) {
            throw new Error(`Error fetching JWKS from jku: ${error.message}`);
        }

        if (!jwks || !Array.isArray(jwks.keys)) {
            throw new Error('Invalid JWKS: Expected keys array');
        }

        const jwk = jwks.keys.find((key) => key.kid === kid);
        if (!jwk) {
            throw new Error('Invalid token: kid not found in JWKS');
        }

        if (jwk.alg !== 'RS256') {
            throw new Error('Invalid key algorithm: Expected RS256');
        }

        if (!jwk.n || !jwk.e) {
            throw new Error('Invalid JWK: Missing modulus (n) or exponent (e)');
        }

        const publicKey = jwkToPem(jwk);

        const decoded = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
        return decoded;
    } catch (error) {
        console.error(`Token verification failed: ${error.message}`);
        throw error;
    }
};
```

This code verifies the JWT token used for authencation. The JWT has a header named `jku`, which stands for JSON Web Key Set URL. The `jku` is a URL that the backend can fetch keys from to then use to sign the JWT token. Thus, in order to prevent arbitrary JWT token forgery, it is important that the JKU is only ever set to a URL that the site owner trusts.

However, in this case, the server only checks that the JKU URL starts with `http://127.0.0.1:1337/`, which leaves room for potential manipulation. While I've previously exploited weak JKU verification by abusing the [@ symbol](https://superuser.com/questions/1578493/whats-the-purpose-of-at-symbol-in-url-before-domain), in this case, that attack isn't valid due to the trailing slash requirement. However, the code here does still allow you to set the JKU to any path on the `127.0.0.1:1337` domain. This ties into the second vulnerability.

`challenge/server/routes/analytics.js`:
```js
import { trackClick, getAnalyticsData } from '../services/analyticsService.js';

export default async function analyticsRoutes(fastify) {
    fastify.get('/redirect', async (req, reply) => {
        const { url, ref } = req.query;

        if (!url || !ref) {
            return reply.status(400).send({ error: 'Missing URL or ref parameter' });
        }
        // TODO: Should we restrict the URLs we redirect users to?
        try {
            await trackClick(ref, decodeURIComponent(url));
            reply.header('Location', decodeURIComponent(url)).status(302).send();
        } catch (error) {
            console.error('[Analytics] Error during redirect:', error.message);
            reply.status(500).send({ error: 'Failed to track analytics data.' });
        }
    });

    fastify.get('/data', async (req, reply) => {
        const { start = 0, limit = 10 } = req.query;

        try {
            const analyticsData = await getAnalyticsData(parseInt(start), parseInt(limit));
            reply.send(analyticsData);
        } catch (error) {
            console.error('[Analytics] Error fetching data:', error.message);
            reply.status(500).send({ error: 'Failed to fetch analytics data.' });
        }
    });
}
```

The comment confirms that the server features an open redirect endpoint. In other words, by navigating to the `/api/analytics/redirect` endpoint, my request can be redirected to anywhere of my choosing. This functionality can be exploited in combination with the first vulnerability to set the JKU to a server that I control. Setting the JKU to something like `http://127.0.0.1:1337/api/analytics/redirect?url=http%3A%2F%example.com&ref=a` passes the JKU verification check, but redirects my request to an unauthorized location.

As a result, by setting up my own publicly accessible JWKS (JSON Web Key Set) and setting it as the JKU, I can create any JWT token I want using my own private/public key pair that will be successfully verified by the server.

My goal was to create my own file in the same format as what was returned by the actual JWKS, but with my own public key (n) instead.

![jwks format](jwks_format.png)
_Format of actual key file from server_

First, I generated my own key pair:

```shell
openssl genpkey -algorithm RSA -out private.pem -pkeyopt rsa_keygen_bits:2048
openssl rsa -in private.pem -pubout -out public.pem
```

I then set up a file on a public webserver that would return the proper json data, but with my own public key data instead. Then, using [jwt.io](https://jwt.io), I forged my own token as the financial controller user.

Lastly, to get the flag, I needed to exploit the third vulnerability (below) to bypass the OTP check (required to send a transaction).

`challenge/server/middleware/otpMiddleware.js`:
```js
import { hgetField } from '../utils/redisUtils.js';

export const otpMiddleware = () => {
  return async (req, reply) => {
    const userId = req.user.email;
    const { otp } = req.body;

    const redisKey = `otp:${userId}`;
    const validOtp = await hgetField(redisKey, 'otp');

    if (!otp) {
      reply.status(401).send({ error: 'OTP is missing.' });
      return
    }

    if (!validOtp) {
      reply.status(401).send({ error: 'OTP expired or invalid.' });
      return;
    }

    // TODO: Is this secure enough?
    if (!otp.includes(validOtp)) {
      reply.status(401).send({ error: 'Invalid OTP.' });
      return;
    }
  };
};
```

This code notably only checks that the otp *includes* a valid OTP. As a result, by changing the JSON data to pass a list of all possible valid OTPs instead, we can bypass this check, thereby earning us the flag.
