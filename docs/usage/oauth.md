# Performing OAuth

Once the library is set up for your project, you'll be able to use it to start adding functionality to your app. The first thing your app will need to do is to obtain an access token to the Admin API by performing the OAuth process.

To do that, you can follow the steps below.

## Add a route to start OAuth

The route for starting the OAuth process (in this case `/login`) will use the library's `beginAuth` method.  The `beginAuth` method takes in the request and response objects (from the `http` module), along with the target shop _(string)_, redirect route _(string)_, and whether or not you are requesting [online access](https://shopify.dev/concepts/about-apis/authentication#api-access-modes) _(boolean)_.  The method will return a URI that will be used for redirecting the user to the Shopify Authentication screen.

### Node.js

```typescript
  :
  } // end of if(pathName === '/')

  if (pathName === '/login') {
    // process login action
    try {
      const authRoute = await Shopify.Auth.OAuth.beginAuth(request, response, SHOP, '/auth/callback');

      response.writeHead(302, { 'Location': authRoute });
      response.end();
    }
    catch (e) {
      console.log(e);

      response.writeHead(500);
      if (e instanceof Shopify.Errors.ShopifyError) {
        response.end(e.message);
      }
      else {
        response.end(`Failed to complete OAuth process: ${e.message}`);
      }
    }
    return;
  } // end of if (pathName === '/login')
} // end of onRequest()

http.createServer(onRequest).listen(3000);
```

### Express

```ts
app.get('/login', async (req, res) => {
  let authRoute = await Shopify.Auth.OAuth.beginAuth(req, res, SHOP, '/auth/callback', true);
  return res.redirect(authRoute);
})
```

## Add your OAuth callback route

After the app is authenticated with Shopify, the Shopify platform will send a request back to your app using this route (which you provided as a parameter to `beginAuth`, above). Your app will now use the provided `validateAuthCallback` method to finalize the OAuth process. This method _has no return value_, so you should `catch` any errors it may throw.

### Node.js

```typescript
  } // end of if (pathName === '/login')

  if (pathName === '/auth/callback') {
    try {
      await Shopify.Auth.OAuth.validateAuthCallback(request, response, query as AuthQuery);

      // all good, redirect to '/'
      response.writeHead(302, { 'Location': '/' });
      response.end();
    }
    catch (e) {
      console.log(e);

      response.writeHead(500);
      if (e instanceof Shopify.Errors.ShopifyError) {
        response.end(e.message);
      }
      else {
        response.end(`Failed to complete OAuth process: ${e.message}`);
      }
    }
    return;
  } // end of if (pathName === '/auth/callback'')
}  // end of onRequest()

http.createServer(onRequest).listen(3000);
```

### Express

```ts
app.get('/auth/callback', async (req, res) => {
  try {
    await Shopify.Auth.OAuth.validateAuthCallback(req, res, req.query as unknown as AuthQuery); // req.query must be cast to unkown and then AuthQuery in order to be accepted
  } catch (error) {
    console.error(error); // in practice these should be handled more gracefully
  }
  return res.redirect('/'); // wherever you want your user to end up after OAuth completes
});
```

After process is completed, you can navigate to `{your ngrok address}/oauth/begin` in your browser to begin OAuth. When it completes, you will have a Shopify session that enables you to make requests to the Admin API, as detailed next.

You can use the `Shopify.Utils.loadCurrentSession()` method to load an online session automatically based on the current request. It will use cookies to load online sessions for non-embedded apps, and the `Authorization` header for token-based sessions in embedded apps, making all apps safe to use in modern browsers that block 3rd party cookies.

[Back to guide index](../index.md)