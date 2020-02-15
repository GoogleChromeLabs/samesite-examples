<!--
 Copyright 2019 Google Inc.

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->

# PHP example for `SameSite=None; Secure`

As of PHP 7.3.0 the
[`setcookie()`](https://www.php.net/manual/en/function.setcookie.php) method
supports the `SameSite` attribute in its options and will accept `None` as a
valid value.

```php
setcookie('same-site-cookie', 'foo', ['samesite' => 'Lax']);
setcookie('cross-site-cookie', 'bar', ['samesite' => 'None', 'secure' => true]);
```

For earlier versions of PHP, you can also set the
[`header()`](https://www.php.net/manual/en/function.header.php) directly:

```php
header('Set-Cookie: same-site-cookie=foo; SameSite=Lax');
header('Set-Cookie: cross-site-cookie=bar; SameSite=None; Secure');
```

For Session Cookie , you can set into [`session_set_cookie_params`](https://www.php.net/manual/en/function.session-set-cookie-params.php)  method.
PHP 7.3.0 introduced new attributes for samesite.

```php
if (PHP_VERSION_ID < 70300) { 
session_set_cookie_params([
    'lifetime' => $cookie_timeout,
    'path' => '/',
    'domain' => $cookie_domain,
    'secure' => $session_secure,
    'httponly' => $cookie_httponly,
    'samesite' => 'Lax'
]);
} else { 
session_set_cookie_params([
    'lifetime' => $cookie_timeout,
    'path' => '/; samesite=Lax',
    'domain' => $cookie_domain,
    'secure' => $session_secure,
    'httponly' => $cookie_httponly
]);
}
```
