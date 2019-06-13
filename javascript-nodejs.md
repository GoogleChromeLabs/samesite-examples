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

# Node.js example for `SameSite=None; Secure`

The most popular library for cookie management in Node.js is the appropriately
named [`cookie` package](https://www.npmjs.com/package/cookie). As of version
0.3.1 it supports the `SameSite` attribute, and as of version 0.4.0 it supports
the `None` value.

```js
response.cookie('same-site-cookie', 'foo', { sameSite: 'lax' });
response.cookie('cross-site-cookie', 'bar', { sameSite: 'none', secure: true });
```

If you are depending on an earlier version, you will need to send the
`Set-Cookie` header directly using
[`response.setHeader()`](https://nodejs.org/docs/v0.4.0/api/http.html#response.setHeader).
Remember, calling this will overwrite anything you may have set earlier in the
process so you will need to set all your cookies here.

```js
response.setHeader('set-cookie', [
  'same-site-cookie=bar; SameSite=Lax',
  'cross-site-cookie=foo; SameSite=None; Secure',
]);
```
