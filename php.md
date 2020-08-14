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
// Set a same-site cookie for first-party contexts
setcookie('cookie1', 'value1', ['samesite' => 'Lax']);
// Set a cross-site cookie for third-party contexts
setcookie('cookie2', 'value2', ['samesite' => 'None', 'secure' => true]);
```

For earlier versions of PHP, you can also set the
[`header()`](https://www.php.net/manual/en/function.header.php) directly:

```php
// Set a same-site cookie for first-party contexts
header('Set-Cookie: cookie1=value1; SameSite=Lax', false);
// Set a cross-site cookie for third-party contexts
header('Set-Cookie: cookie2=value2; SameSite=None; Secure', false);
```

For Session Cookie , you can set into [`session_set_cookie_params`](https://www.php.net/manual/en/function.session-set-cookie-params.php)  method.
PHP 7.3.0 introduced new attributes for samesite.

```php
if (PHP_VERSION_ID >= 70300) { 
session_set_cookie_params([
    'lifetime' => $cookie_timeout,
    'path' => '/',
    'domain' => $cookie_domain,
    'secure' => $session_secure,
    'httponly' => $cookie_httponly,
    'samesite' => 'Lax'
]);
} else { 
session_set_cookie_params(
    $cookie_timeout,
    '/; samesite=Lax',
    $cookie_domain,
    $session_secure,
    $cookie_httponly
);
}
```


If you need to set SameSite=None, it's worth checking for incompatible clients, and setting cookies without the SameSite rule for those:

```php
function hasWorkingSameSiteNoneCookies()
{
    if(!isset($_SERVER['HTTP_USER_AGENT']))
    {
        return true;
    }

    $ua = $_SERVER['HTTP_USER_AGENT'];

    //ios 12 iphones/ipods
    if(strpos($ua, 'CPU iPhone OS 12') !== false)
    {
        return false;
    }

    //ios 12 ipads
    if(strpos($ua, 'iPad; CPU OS 12') !== false)
    {
        return false;
    }

    //MacOS 10.14 safari
    if(
        strpos($ua, 'Macintosh; Intel Mac OS X 10_14') !== false && strpos($ua, 'Version/') !== false
        && strpos($ua, 'Safari') !== false
    )
    {
        return false;
    }

    //chrome 50 - 69 (more versions than necessary, but that does not matter)
    if(strpos($ua, 'Chrome/5') !== false || strpos($ua, 'Chrome/6') !== false)
    {
        return false;
    }

    return true;
}
```
