# Token API - Client side

The Token API let you authorize your user with one of our 120+ providers. The most known example is to add a Facebook connect to your website to ease the emboarding of your user.

You can get this authorization client-side or server-side depending your needs.

<div class="code-block Node"><pre>The Client side section is not available for this SDK</pre></div>

<aside class="notice">The client side authorization can be made only by one of these client-side SDKs: Javascript, Phonegap, iOS, Android</aside>

## Configuration

To authorize your users using OAuth.io, you just need to add a provider to your OAuth.io app and copy paste your provider's API Keys. Once added, you can try if the provider works by clicking on the `Try auth` button.

## Authorize with popup

<div class="code-block Javascript Phonegap"><pre><code class="highlight javascript">//Example with Facebook
OAuth.popup('facebook').done(function(facebook) {
  //make API calls with `facebook`
}).fail(function(err) {
  //todo when the OAuth flow failed
});</code></pre>
<pre><code class="highlight javascript">//Example with Twitter with the cache option enabled
OAuth.popup('twitter', {cache: true}).done(function(twitter) {
  //make API calls with `twitter`
}).fail(function(err) {
  //todo when the OAuth flow failed
})</code></pre>
</div>

This mode ask the user's authorization in a simple popup. You can send multiple option to customize it.

Options|Description|Type|Default value
-------|-----------|----|-------------
cache|If set to true, when the popup is called, it will directly return the cached credentials (if available) instead of showing a popup everytime the user logs in. That is to say, once the user has seen the popup once, his credentials are kept in the browser local storage.|boolean|false
authorize|Some OAuth provider let developers send parameters to customer the authorization popup|Object|null

## Authorize with redirection

<div class="code-block Javascript Phonegap">
  <pre><code class="highlight javascript">//Example with Google
OAuth.redirect('google', 'http://localhost/callback');</code></pre>
  <pre><code class="highlight javascript">//in your callback page (can be the same page)
OAuth.callback('google').done(function(google) {
  //make API calls with `google`
}).fail(function(err) {
  //todo when the OAuth flow failed
})</code></pre>
</div>

Redirects to the provider's login page, where the user can accept your app's permissions. Once the user accepts the permissions, they are redirected to the callback URL.

<br style="clear:both">

## Cache the result

Client-side SDKs allow to cache the result to not have to show the popup everytime you need the access token.

<div class="code-block Javascript Phonegap">
  <pre><code class="highlight javascript">//Example with Twitter with the cache option enabled
OAuth.popup('twitter', {cache: true}).done(function(twitter) {
  //make API calls with `twitter`
}).fail(function(err) {
  //todo when the OAuth flow failed
})</code></pre>
</div>

### Save your user's authorization in the cache

If the cache option is enabled, when the popup is called, it will directly return the cached OAuth result (if available) instead of showing a popup everytime the user logs in. That is to say, once the user has seen the popup once, his credentials are kept in the browser local storage for Javascript/Phonegap SDK and in the mobile with iOS and Android.

<div class="code-block Javascript Phonegap">
  <pre><code class="highlight javascript">var twitter = OAuth.create('twitter');
//`twitter` is an OAuth result.
//`twitter` can be `null` if the OAuth result has not been created yet.</code></pre>
</div>

### Create an OAuth result from Cache

Once an OAuth result has been cached, you can re-create the OAuth result from it (later or in another page for instance).

Note that you can also create an OAuth result from existing credentials if needed.

<div class="code-block Javascript Phonegap">
  <pre><code class="highlight javascript">//clear the cache for Facebook
OAuth.clearCache('facebook');
//remove all the cache
OAuth.clearCache();</code></pre>
</div>

### Clear the cache

You can of course remove partially or totally the cache generated by OAuth.io. Note that it can be used when you want to logout your user to be sure it will open the popup the next time the user try to login.


# Token API - Server side

The server side authorization is mostly use to get a refresh token and make action on behalf of your user when they are not online on your website (for instance, crawl your user's feed and update them everyday even if the user is not connected).

## Configuration

As for the client side authorization, you need to add a provider to your OAuth.io app and copy paste your provider's API keys. But this time, you have to select a backend in your oauth.io app, like that, only server side authorization will be allowed.

You can also set the both mode to get a copy of the access_token client side too (and use all of the results method).

## Redirect

<div class="code-block Node">
  <pre><code class="highlight javascript">//syntaxe
app.get(authorizeEndpoint, OAuth.auth(provider, urlToRedirect));
app.get(redirectEndpoint, OAuth.redirect(function(result, req, res) {
    //todo with result
}));

//exemple with Linkedin
app.get('/signin', OAuth.auth('linkedin', 'http://localhost:8080/oauth/redirect'));

app.get('/oauth/redirect', OAuth.redirect(function(result, req, res) {
    if (result instanceof Error) {
        res.send(500, "error: " + result.message);
    }
    result.me().done(function(me) {
        console.log(me);
        res.send(200, JSON.stringify(me));
    });
}));</code></pre>
</div>

<div class="code-block Javascript iOS Android">
  <pre>This feature is not supported by this SDK</pre>
</div>

This method is the simplest way to achieve a server side authorization.

You need to define 2 endpoints:

- the first endpoint (`authorizeEndpoint`) is where you will redirect your users to authorize them to one of our 120+ OAuth `provider`

- then they will be redirected to the second endpoint (`redirectEndpoint`) with the `result` of the authorization in the callback.

In the HTML of your webapp, you just have to create a link to the first endpoint to start the authorization flow `<a href="/signin">Signin with </a>`

If an error occured, the error is placed in the `result`.

<br style="clear:both;">

## Authorize the user from a frontend SDK

<aside class="info">This authorization can be done by any backend (Ruby, Python, C# etc...) using REST.</aside>

This method is a bit longer than the redirect one but can be used by every backend. This can be done in 3 steps:

<div class="code-block Node"><pre><code class="highlight javascript">//Node.js
var token = OAuth.generateStateToken(req);
res.send(200, {token:token});</code></pre>
</div>

* Generate a state token in your backend. Basically, it generate a unique id, store it in session and send it to the front.

<br style="clear:both;">
<div class="code-block Node PHP Go Javascript"><pre><code class="highlight javascript">//Javascript (client-side)
OAuth.popup(provider, {
  state: params.token
}).done(function(result) {
  $.post('/auth', {code: result.code})
})
</code></pre></div>

* Authorize your user with a client side SDK using the state token (you will receive a code instead of an access_token). After receiving the access token, if you don't use a backend SDK, you need to check manually that the state token have not changed from the session.

<br style="clear:both;">
<div class="code-block Node">
  <pre><code class="highlight javascript">//Node.js
app.get('/auth', function(req, res) {
  OAuth.auth(provider, req.session, {
    code: JSON.parse(req.body).code
  }).then(function(oauthResult) {
    //todo with oauthResult
    //oauthResult.access_token oauthResult.refresh_token
  })
});</code></pre>
</div>

<div class="code-block Javascript iOS Android">
  <pre>POST https://oauth.io/auth/facebook
Body:
  code=ePLi3...EQfdq
  key=public_key
  secret=secret_key</pre>
</div>

* From the backend, send the code to OAuth.io to get back the access_token and refresh_token. OAuth.io will also send back the state token that you have to manually check if it is the same state token than the one stored in session in the 1st step -- **This is automatically done using a server-side SDK**.

## OAuth result from session

<div class="code-block Javascript iOS Android">
  <pre>This feature is not supported by this SDK</pre>
</div>

<div class="code-block Node">
  <pre><code class="highlight javascript">OAuth.auth('facebook', req.session)
  .then(function (facebook) {
    //todo with `facebook`
  });
});</code></pre>
</div>

Once a user has been authorized by a provider, he is stored in session by the SDK that mean you can recreate the OAuth result from the session. If the OAuth Result has an `expires_in` field and the access token has expired, he will automatically refresh the `access_token`.

## Refresh the access token

<div class="code-block Javascript iOS Android">
  <pre># Using a backend
https://oauth.io/auth/refresh_token/:provider
Body:
  token: REFRESH_TOKEN
  key: PUBLIC_OAUTHIO_KEY
  secret: SECRET_OAUTHIO_KEY</pre>
</div>
<div class="code-block Node">
  <pre><code class="highlight javascript">OAuth.refreshCredentials(oauthResult, req.session)
    .then(function(newOAuthResult) {
      //todo with newOAuthResult
    })
    .fail(function(e) {
      //handle an error
    });
});</code></pre>
  <pre><code class="highlight javascript">//or from session
OAuth.auth('facebook', req.session, {
  force_refresh: true
})</code></pre>
</div>

For most providers, the server side authorization without any configuration send back a refresh token. The refresh token is added to the result on a server side authorization only.

This refresh token can be used when an access token expires to regenerate one without having to open a popup and ask your user for permissions (as the user has already accepted them).

In all our server side SDK, a method is available to refresh the access token easily. It gives back a new OAuth object containing a fresh access token.

For Google, to get a refresh token, follow this Stackoverflow link: [http://stackoverflow.com/questions/23759138/getting-refresh-tokens-from-google-with-oauth-io](http://stackoverflow.com/questions/23759138/getting-refresh-tokens-from-google-with-oauth-io)

# OAuth result

<div class="code-block"><pre><code class="highlight json">with Facebook (OAuth 2)
{
  "access_token": "CAAHv0hYN...RmO4zcd",
  "expires_in": 5183561,
  "provider": "facebook"
}</code></pre>
<pre><code class="highlight json">with Twitter (OAuth 1.0a)
{
  "oauth_token": "9568514...0AQMedh",
  "oauth_token_secret": "Ktakg008...60KSNG4",
  "provider": "twitter"
}</code></pre>
<pre><code class="highlight json">with Google using the server side flow with a frontend SDK
{
  "code": "XsrpW...leE3",
  "provider": "google"
}</code></pre>
<pre><code class="highlight json">with Github using the server side flow in a backend SDK
{
  "provider": "github",
  "access_token": "akpWg0...QEof",
  "refresh_token": "HvOiruf...Zriv",
  "expires_in": 5183561
}</code></pre>
</div>

We call OAuth Result the result object after an authorization. It lets you access the tokens, expires date etc. but also give you simple method to access others OAuth.io API (Make authorized API call to the provider, retrieve unified user data, and more). The OAuth Result will contains these fields:

Field|Description|Type|Example value
-----|-----------|----|-------------
access_token|**OAuth 2** -- The authorization key to access the OAuth2 API|string|CAAHv...aAWy
oauth_token|**OAuth 1** -- The authorization key to access the OAuth1 API|string|XVQpX...WR0K
oauth_token_secret|**OAuth 1** -- The second authorization key to access OAuth1 API|string|PHQD2...V7xw
expires_in|*Optional depending the provider* -- The expiration of the `access_token`|integer|5184000
code|**Client side only** -- The code to be exchanged against the access token **server side authorization**|string|XsrpW...leE3
refresh_token|**Server side only** -- The refresh token is used to refresh the access_token to extend the expiration.|string|Tgfgso|...Geo4e5
provider|The provider your user is connected with|string|facebook

The access token can be sent to your backend to make API calls but, you can't be sure that the access token is a real one (as the user could fake a request with a wrong access token) so you need to make a test API call and see if it returns 200 (OK) or 401 (UNAUTHORIZED).

As you can see, using the client side authorization, you won't have a `refresh_token`. For this, take a look at the `Server side authorization` section

<aside class="notice">OAuth.io SDKs will extend the result with some usefull methods to access others OAuth.io APIs:

* `get()` `post()` `put()` `del()` `patch()` - Request API

* `me()` - User Data API

* more soon

</aside>