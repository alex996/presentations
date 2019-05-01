# express-session

A quick guide for setting up and configuring [express-session](https://www.npmjs.com/package/express-session) middleware.

```js
const express = require('express')

// Sessions can be stored server-side (ex: user auth) or client-side
// (ex: shopping cart). express-session saves sessions in a store, and
// NOT in a cookie. To store sessions in a cookie, use cookie-session.
const session = require('express-session')

// The default in-memory store is not production-ready because it leaks
// memory and doesn't scale beyond once process. For production, we need
// a session store, like Redis, which we can wire up with connect-redis.
const RedisStore = require('connect-redis')(session)

const app = express()

// Redis is a key-value NoSQL data store. It's perfect for sessions,
// because they can be serialized to JSON and stored temporarily using
// SETEX to auto-expire (i.e. auto-remove) after maxAge in seconds.
const store = new RedisStore({
  host: 'localhost', port: 6379, pass: 'secret'
})

app.use(
  // Creates a session middleware with given options.
  session({

    // Defaults to MemoryStore, meaning sessions are stored as POJOs
    // in server memory, and are cleared out when the server restarts.
    store,

    // Name for the session ID cookie. Defaults to 'connect.sid'.
    name: 'sid',

    // Whether to force-save unitialized (new, but not modified) sessions
    // to the store. Defaults to true (deprecated). For login sessions, it
    // makes no sense to save empty sessions for unauthenticated requests,
    // because they are not associated with any valuable data yet, and would
    // waste storage. We'll only save the new session once the user logs in.
    saveUninitialized: false,

    // Whether to force-save the session back to the store, even if it wasn't
    // modified during the request. Default is true (deprecated). We don't
    // need to write to the store if the session didn't change.
    resave: false,

    // Whether to force-set a session ID cookie on every response. Default is
    // false. Enable this if you want to prolong session lifetime while the user
    // is still browsing the site. Beware that the module doesn't have an absolute
    // timeout option, so you'd need to handle indefinite sessions manually.
    // rolling: false,

    // Secret key to sign the session ID. The signature is used
    // to validate the cookie against any tampering client-side.
    secret: 'sssh, quiet! it\'s a secret!',

    // Settings object for the session ID cookie. The cookie holds a
    // session ID ref in the form of 's:{SESSION_ID}.{SIGNATURE}' as in
    // s%3A9vKnWqiZvuvVsIV1zmzJQeYUgINqXYeS.nK3p01vyu3Zw52x857ljClBrSBpQcc7OoDrpateKp%2Bc

    // It is signed and URL encoded, but NOT encrypted, because session ID is
    // merely a random string that serves as a reference to the session. Even
    // if encrypted, it still maintains a 1:1 relationship with the session.
    // OWASP: cookies only need to be encrypted if they contain valuable data.
    // See https://github.com/expressjs/session/issues/468

    cookie: {

      // Path attribute in Set-Cookie header. Defaults to the root path '/'.
      // path: '/',

      // Domain attribute in Set-Cookie header. There's no default, and
      // most browsers will only apply the cookie to the current domain.
      // domain: null,

      // HttpOnly flag in Set-Cookie header. Specifies whether the cookie can
      // only be read server-side, and not by JavaScript. Defaults to true.
      // httpOnly: true,

      // Expires attribute in Set-Cookie header. Set with a Date object, though
      // usually maxAge is used instead. There's no default, and the browsers will
      // treat it as a session cookie (and delete it when the window is closed).
      // expires: new Date(...)

      // Preferred way to set Expires attribute. Time in milliseconds until
      // the expiry. There's no default, so the cookie is non-persistent.
      maxAge: 1000 * 60 * 60 * 2,

      // SameSite attribute in Set-Cookie header. Controls how cookies are sent
      // with cross-site requests. Used to mitigate CSRF. Possible values are
      // 'strict' (or true), 'lax', and false (to NOT set SameSite attribute).
      // It only works in newer browsers, so CSRF prevention is still a concern.
      sameSite: true,

      // Secure attribute in Set-Cookie header. Whether the cookie can ONLY be
      // sent over HTTPS. Can be set to true, false, or 'auto'. Default is false.
      secure: process.env.NODE_ENV === 'production'
    }
  })
)

app.get('/', (req, res) => {
  // A new uninitialized session is created for each request (but not
  // persisted to the store if saveUninitialized is false). It's
  // automatically serialized to JSON and can look something like
  /*
    Session {
      cookie: {
        path: '/',
        _expires: 2018-11-18T01:33:01.043Z,
        originalMaxAge: 7200000,
        httpOnly: true,
        sameSite: true,
        secure: false
      }
    }
  */
  console.log(req.session)

  // You can also access the cookie object above directly with
  console.log(req.session.cookie)

  console.log(req.session.cookie.expires) // date of expiry
  console.log(req.session.cookie.maxAge) // milliseconds left until expiry

  // Unless a valid session ID cookie is sent with the request,
  // the session ID below will be different for each request.
  console.log(req.session.id) // ex: VdXZfzlLRNOU4AegYhNdJhSEquIdnvE-

  // Same as above. Alphanumeric ID that gets written to the cookie.
  // It's also SESSION_ID porton in 's:{SESSION_ID}.{SIGNATURE}'.
  console.log(req.sessionID)

  res.send(`<a href='/login'>Login</a>`)
})

app.get('/login', (req, res) => {
  // Unless we explicitly write to the session (and resave is false), the
  // store is never updated, even though a new session is generated on each
  // request. After we modify that session, it gets persisted. On subsequent
  // writes, its cookie.maxAge also gets updated and synced with the store.
  req.session.userId = 1

  // Note that userId has no special meaning. It makes sense to store a
  // unique reference, e.g. user ID, because we can then use it to look up
  // the user in the DB. But we could store anything in sessions.

  res.send(`
    <form method='post' action='/logout'>
      <button>Logout</button>
    </form>
  `)

  // By the end of each request, the session middleware will call session.touch()
  // to update maxAge (and thus expires). It might call session.save() to write
  // to the store but ONLY IF the data was altered manually, like above (with
  // the exception when resave is set to true).
})

app.post('/logout', (req, res) => {
  // Assuming the request was authenticated in /login above,
  /*
    Session {
      cookie: {
        path: '/',
        _expires: 2018-11-18T02:03:01.926Z,
        originalMaxAge: 7200000,
        httpOnly: true,
        secure: false,
        sameSite: true
      },
      userId: 1 <-- userId from /login
    }
  */
  console.log(req.session)

  console.log(req.session.id) // ex: 0kVkUn7KUX1UZGnjagDKd_NPerjXKJsA

  // Note that the portion between 's%3A' and '.' is the session ID above.
  // 's%3A' is URL encoded and decodes to 's:'. The last part is the signature.
  // sid=s%3A0kVkUn7KUX1UZGnjagDKd_NPerjXKJsA.senfzYOeNHCtGUNP4bv1%2BSdgSdZWFtoAaM73odYtLDo
  console.log(req.get('cookie'))

  // Upon logout, we can destroy the session and unset req.session.
  req.session.destroy(err => {
    // We can also clear out the cookie here. But even if we don't, the
    // session is already destroyed at this point, so either way, they
    // won't be able to authenticate with that same cookie again.
    res.clearCookie('sid')

    res.redirect('/')
  })
})

app.listen(3000, () => console.log('http://localhost:3000'))

```
