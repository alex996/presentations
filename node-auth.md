# Authentication in Node.js

## Features

- [x] login/logout/register + session expiry
- [x] email verification (`"Confirm your email"`)
- [x] password reset (`"Forgot password"`)
- [x] password confirmation (`"Re-enter your password"`)
- [x] persistent login (`"Remember me"`)
- [x] account lockout (`"Too many failed login attempts"`)
- [x] rate limiting (`"Too many requests"`)

## Theory

### Authentication

- verifying user identity
- public (username/email) + private (pwd/token/2FA) info

### Session Management

- HTTP is a stateless protocol; each req is self-contained
- sessions used to retain user state between requests
- session cookie ties a request to the user's session

### Session Timeout

- idle/inactivity: sliding expiration, resets on each request
- absolute: fixed expiration, max duration of lifetime
- renewal: interval until session ID is regenerated

### Cryptography

- hashing: fast & deterministic one-way transformation from plaintext to a digest
  - keyless: message only, e.g. `MD5`, `SHA1`, `SHA256`, `SHA512`
  - keyed: message + secret key, e.g. `HMAC SHA256`
- encryption: two-way transformation from plaintext to ciphertext
  - symmetric: one key, e.g. `AES`
  - asymmetric: public + private key pair, e.g. `RSA`

### Encoding

- encoding: reversible transformation without any keys
  - e.g. `base64`, `base64url`, `hex`
- compression: encoding that reduces original size
  - e.g. `gzip`, `brotli`

### Formats

- base64 `[A-Za-z0-9+/=]`
  - encodes 3 bytes into 4 chars, `ceil(n / 3) * 4`
  - e.g. `20 bytes -> ceil(20 / 3) * 4 = 7 * 4 = 28 chars`
  - not URL-safe:  (use `base64url`)
- hex/base16 `[a-f0-9]`
  - encodes 1 byte into 2 chars, `n * 2`
  - e.g. `20 bytes -> 20 * 2 = 40 chars`

## Passwords

### Flow

1. server runs a plaintext pwd through a **KDF** (_key derivation function_)
2. KDF appends a unique **salt** (often auto-generated) & produces a **hash**
3. hash + salt (in plaintext) is stored in the password field in DB
   - salt is not secret, only used against pre-computed rainbow tables
4. when user re-enters the pwd, server runs it through the same hash func
   - KDF extracts the salt, re-generates the hash, performs comparison

### Caveats

- given unique salts, two identical passwords produce different hashes
  - attacker would need a rainbow table for each salt
- once hashed, original pwd is discarded; hash cannot be reversed
- fast hashes (`SHA1`, `SHA256`, etc.) are not suitable
  - passwords lack entropy (short, weak), vulnerable to brute-force
  - HMAC/SHA hashes are extremely fast to compute with cheap GPU
  - KDFs run a pwd through many rounds of hashing
    - adaptive work factor safeguards against advances in computing

## Password Hashing

### Objectives

Slow hash function, takes many iterations, designed against brute forcing

- CPU-intensive
  - resilient to GPU acceleration
- RAM-intensive
  - limit parallelism

### Algorithms/KDFs

1. argon2
2. bcrypt
3. scrypt
4. pbkdf2

### Bcrypt

Truncates input string

- on a null byte `0x00` (in C/C++)
- after 72 bytes (in `utf8`)

## Password Storage

### Attacks

- rainbow table attack: using a lookup table to derive original pwd from its hash
- dictionary attack: trying common passwords from a dict
- brute force attack: trying all possible password combinations

### Salt & Pepper

- salt: unique string appended to pwd before hashing
  - not secret, stored in plaintext in DB, thwarts rainbow table attacks
- pepper: secret salt, i.e. key (either appended or signed with, e.g. HMAC)
  - NOT stored in DB, only on the server; slows down brute-force attacks

### Approaches

1. hash `bcrypt(passphrase, salt)`
2. pre-hash `bcrypt(sha256(passphrase).base64(), salt)`
   - sha256/sha512 digest may contain null bytes (end of string)
   - do NOT use raw binary; wrap with base64
3. pre-hash with a key (pepper) `bcrypt(hmac_sha256(passphrase, key).base64(), salt)`
4. hash & encrypt `aes256(bcrypt(passphrase, salt), key, iv)`
5. pre-hash & encrypt `aes256( bcrypt(sha256(passphrase).base64(), salt), key, iv )`

## Email Verification/Confirmation

### Flow

1. user signs up on the website
2. server generates & signs an activation link
   - link expires after X hours/days
3. servers sends an email with the link
4. user visits the link & verifies their account
5. (optional) user requests the link to be resent
   - if link hasn't expired, resend
   - else, generate & email a new link

### Requirements

- token is not (easily) predictable
- URL is signed to guard against forgery
- older email links are [valid until expiry](https://github.com/laravel/framework/issues/27644)

### Implementations

- **Laravel**

> temporary (absolute) signed URL (HMAC SHA256, not persisted)

```php
$parameters = ['id' => $user->id, 'hash' => sha1($user->email), 'expires' => Carbon::now()->addMinutes(60)];

return $this->route('verification.verify', $parameters + [
  'signature' => hash_hmac('sha256', $this->routeUrl()->to($route, $parameters, $absolute), $key),
], $absolute);

// e.g. http://example.com/email/verify/{user.id}/{sha1(user.email)}?expires=1521543365&signature=d32f53ced4a781f287b612d21a3b7d3c38ebc5ae53951115bb9af4bc3f88a87a
```

See [`Illuminate/Routing/UrlGenerator.php`](https://github.com/laravel/framework/blob/5b1b3675748649da19c9b6308d1ade25f41eabd5/src/Illuminate/Routing/UrlGenerator.php#L321), [`Illuminate/Auth/Notifications/VerifyEmail.php`](https://github.com/laravel/framework/blob/7b074c1ac506c1895f85ec77481b55228a122a05/src/Illuminate/Auth/Notifications/VerifyEmail.php#L61), and [`Illuminate/Foundation/Auth/VerifiesEmails.php`](https://github.com/laravel/framework/blob/086c02a544af2870c52976408cded8cf59d7cf40/src/Illuminate/Foundation/Auth/VerifiesEmails.php#L36)

- **Rails/Devise**

  - random(20) token stored in plaintext
    - doesn't log user in after verification
  - re-sends current token, thus same email link
    - if expired (after 3 days), regenerates & saves

```ruby
if self.confirmation_token && !confirmation_period_expired?
  @raw_confirmation_token = self.confirmation_token
else
  self.confirmation_token = @raw_confirmation_token = SecureRandom.urlsafe_base64(20)
  self.confirmation_sent_at = Time.now.utc
end
```

See [`devise/models/confirmable.rb`](https://github.com/plataformatec/devise/blob/43068ac23907cbf8f74dfedd60fe4e9f14560006/lib/devise/models/confirmable.rb#L248)

- **Django/django-registration**

> username -> HMAC SHA1 (not persisted, expires after X day(s))

```python
base64data = base64.urlsafe_b64encode( json.dumps(user.get_username()).encode('latin-1') )

key = hashlib.sha1('registration' + settings.SECRET_KEY).digest()

signature = hmac.new(key, msg=base64data, digestmod=hashlib.sha1)

token = f'{base64data}:{signature}'
```

See [`activation/views.py`](https://github.com/ubernostrum/django-registration/blob/58be01f5858a95d30f99eb618d15363323c5d168/src/django_registration/backends/activation/views.py#L67) and [`core/signing.py`](https://github.com/django/django/blob/b9cf764be62e77b4777b3a75ec256f6209a57671/django/core/signing.py#L93)

## Password Reset/Recovery

### Flow

1. user visits "Forgot Password" page
2. user fills in their email and submits the form
3. server generates a unique & random token
4. server hashes the token and saves the **hash** & **exp. date** in DB
   - if DB is compromised, attacker can't use the hash to reset user pwd
5. server constructs a link with plaintext token and emails the user
6. user visits the link and submits a new password
7. server verifies the token and updates user password

### Tokens

- at least 32 chars long (OWASP)
- generated using a secure PRNG (Pseudorandom number generator)
  - don't use current time, user email/ID
- hashed before stored in DB (tokens = credentials)
- one-time use
- short-lived (has exp. date)

### Deactivation

- when a new one is issued, delete all older tokens
  - simpler DB design, poorer UX
- when the password is reset (either all or expired only)
  - more complex DB design, better UX

### Implementations

- **Laravel**

> random(40) -> HMAC SHA256 -> bcrypt

```php
$token = hash_hmac('sha256', random_bytes(40), app['config']['app.key'])
$dbToken = password_hash($token, PASSWORD_BCRYPT, ['cost' => 10])
```

See [`Illuminate/Auth/Passwords/DatabaseTokenRepository.php`](https://github.com/laravel/framework/blob/ba4204f3a8b9672b6116398c165bd9c0c6eac077/src/Illuminate/Auth/Passwords/DatabaseTokenRepository.php#L214) and [`Illuminate/Hashing/BcryptHasher.php`](https://github.com/laravel/framework/blob/ba4204f3a8b9672b6116398c165bd9c0c6eac077/src/Illuminate/Hashing/BcryptHasher.php#L47)

- **Rails/Devise**

> random(20) -> HMAC SHA256

```ruby
while find_first({ :reset_password_token => enc })
  raw = SecureRandom.urlsafe_base64(20)
  enc = OpenSSL::HMAC.hexdigest('SHA256', key, raw)
```

See [`devise/models/recoverable.rb`](https://github.com/plataformatec/devise/blob/43068ac23907cbf8f74dfedd60fe4e9f14560006/lib/devise/models/recoverable.rb#L90) and [`devise/token_generator.rb`](https://github.com/plataformatec/devise/blob/43068ac23907cbf8f74dfedd60fe4e9f14560006/lib/devise/token_generator.rb#L20)

- **Django**

> user state -> HMAC SHA1 (not persisted)

```python
key = hashlib.sha1( # salt + secret
    "django.contrib.auth.tokens.PasswordResetTokenGenerator" + settings.SECRET_KEY
).digest()

value = str(user.pk) + user.password + str(last_login) + str(timestamp)

# output: 160 bits / 8 bits/byte / 2 = 40/2 = 20
hash = hmac.new(key, msg=value, digestmod=hashlib.sha1).hexdigest()[::2]

return "%s-%s" % (days_since_2001_base36, hash)
```

See [`django/contrib/auth/tokens.py`](https://github.com/django/django/blob/b9cf764be62e77b4777b3a75ec256f6209a57671/django/contrib/auth/tokens.py#L58) and [`django/utils/crypto.py`](https://github.com/django/django/blob/b9cf764be62e77b4777b3a75ec256f6209a57671/django/utils/crypto.py#L39)

## Password Confirmation

- periodically require logged-in user to re-enter their password
- used on sensitive pages, e.g. payment info or orders

### Implementations

- **Laravel**

```php
$confirmedAt = time() - $request->session()->get('auth.password_confirmed_at', 0);
return $confirmedAt > config('auth.password_timeout', 10800);
```

See [laravel/framework#30214](https://github.com/laravel/framework/pull/30214/) and [laravel/laravel#5129](https://github.com/laravel/laravel/pull/5129) PRs

## Persistent Login

### Requirements

- remember users on [multiple devices](https://github.com/laravel/ideas/issues/971)
- sign the cookie to prevent forgery (and possibly encrypt)
- check for cookie expiry [server-side](https://news.ycombinator.com/item?id=5969932)
- once logged out, clear remember me token & [unset the cookie](https://www.reddit.com/r/laravel/comments/8w0y51/laravel_remember_me_not_working_as_expected/)

### Approaches

- extend expiration date (e.g. 2 hours -> 1 week)
- set a remember-me cookie with a signed token
  - re-instate an auth session
  - if detected an anomaly (e.g. pwd reset), unset cookie & return 401

### Implementations

- **Laravel**

> `remember_me` field `VARCHAR(100)` in `users` table

```php
public function login(AuthenticatableContract $user, $remember = false) {
  // ...
  if ($remember) {
    if (empty($user->getRememberToken())) {
      $user->setRememberToken($token = Str::random(60)); # stored in plaintext
    }

    // signed + encrypted, 5 years
    $this->getCookieJar()->forever(
      'remember_'.'session'.'_'.sha1(static::class),
      $user->id.'|'.$user->remember_token().'|'.$user->password()
    )
  }
  // ...
}
```

See [`Illuminate/Foundation/Auth/AuthenticatesUsers.php`](https://github.com/laravel/framework/blob/e04a7ffba8b80b934506783a7d0a161dd52eb2ef/src/Illuminate/Foundation/Auth/AuthenticatesUsers.php#L45), [`Illuminate/Auth/SessionGuard.php`](https://github.com/laravel/framework/blob/e04a7ffba8b80b934506783a7d0a161dd52eb2ef/src/Illuminate/Auth/SessionGuard.php#L412), and [`Illuminate/Auth/DatabaseUserProvider.php`](https://github.com/laravel/framework/blob/1bbe5528568555d597582fdbec73e31f8a818dbc/src/Illuminate/Auth/DatabaseUserProvider.php#L87)

- **Ruby/Devise**

```ruby
while find_first({ :remember_token => token })
  token = SecureRandom.urlsafe_base64(20)

args = [user.id, token, Time.now.utc.to_f.to_s] # UNIX timestamp
```

See [`devise/models/rememberable.rb`](https://github.com/plataformatec/devise/blob/master/lib/devise/models/rememberable.rb)

## References

### Authentication

- [The definitive guide to form-based website auth](https://stackoverflow.com/questions/549/the-definitive-guide-to-form-based-website-authentication)

- [Your Node.js authentication tutorial is (probably) wrong](https://medium.com/hackernoon/your-node-js-authentication-tutorial-is-wrong-f1a3bf831a46)

### Cryptography

- [Hashing vs encryption](https://stackoverflow.com/a/4948393)

- [You wouldn't base64 a password](https://paragonie.com/blog/2015/08/you-wouldnt-base64-a-password-cryptography-decoded)

- [Why SHA/HMAC are not suitable for passwords](https://security.stackexchange.com/a/16817) (also [this](https://security.stackexchange.com/a/133251), TL;DR KDFs are much slower)

### Encoding

- [How does base64 work](https://www.quora.com/How-does-base64-encoding-work)

- [What is the increase in space with base64](https://stackoverflow.com/a/4715480)

- [Is base64 URL-safe](https://stackoverflow.com/a/1374794) (TL;DR use `base64url`)

### Password Hashing

- [How to securely hash passwords](https://security.stackexchange.com/a/31846)

- [Why bcrypt wins over MD5, SHA, AES](https://stackoverflow.com/a/4435888)

### Hashing Algorithms

- [Argon2 vs bcrypt vs scrypt vs pbkdf2 comparison](https://security.stackexchange.com/a/216381)

- [Best password hashing algo in 2018](https://security.stackexchange.com/a/197550) (TL;DR argon2)

### Mixing Hashing & Encryption

- [How secure is bcrypt(sha1(password))](https://security.stackexchange.com/a/183366)

- [Combining bcrypt with other hashes](https://blog.ircmaxell.com/2015/03/security-issue-combining-bcrypt-with.html) (TL;DR don't forget to base64)

### Bcrypt

- [Does bcrypt have maximum password length](https://security.stackexchange.com/a/184090) (TL;DR 72 bytes, not 56)

- [Why does `0x00` make bcrypt weaker](https://crypto.stackexchange.com/a/33336) (TL;DR strings in C terminate on `'\0'`)

### Password Storage

- [Safely storing passwords](https://www.martinstoeckli.ch/hash/en/index.php)

- [How Dropbox stores passwords (2016)](https://news.ycombinator.com/item?id=12548210)

- [Example pre-hashing lib in .NET](https://github.com/BcryptNet/bcrypt.net)

- [Pepper, hash, hmac, or encryption](https://security.stackexchange.com/a/21264)

### Email Confirmation Tokens

- [What should the confirmation token consist of](https://security.stackexchange.com/a/197005)

- [Why tokens are no longer digested in Rails](https://github.com/plataformatec/devise/issues/3640)

- [How Laravel uses signed URLs for email confirmation](https://laravel-news.com/signed-routes)

### Password Reset Tokens

- [How to generate reset tokens](https://security.stackexchange.com/a/213979)

- [Why salting is optional, but hashing isn't](https://security.stackexchange.com/a/86915)

- [Secure Account Recovery Made Simple (2016)](https://paragonie.com/blog/2016/09/untangling-forget-me-knot-secure-account-recovery-made-simple)

- [Why Laravel bcrypt's reset tokens](https://github.com/laravel/framework/pull/16850)

- [How Django password reset works (without DB persistance)](https://www.sjoerdlangkemper.nl/2016/04/07/djangos-reset-password-mechanism/)

### Password Confirmation

- [How password confirmation works in Laravel](https://laravel-news.com/new-password-confirmation-in-laravel-6-2)

### Persistent Login

- [How to securely implement "remember me" feature](https://security.stackexchange.com/a/109439)

- [Login with "Remember Me" Cookies (2015)](https://paragonie.com/blog/2015/04/secure-authentication-php-with-long-term-persistence)

- [How to build a "remember me" (2013)](https://www.troyhunt.com/how-to-build-and-how-not-to-build/)

### OWASP Cheat Sheets

- [Authentication](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Authentication_Cheat_Sheet.md)

- [Session Management](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Session_Management_Cheat_Sheet.md)
