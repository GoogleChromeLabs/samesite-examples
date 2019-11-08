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

# Cloudflare worker example for modifying existing cookies

A rough proof of concept that uses a
[Cloudflare worker](https://developers.cloudflare.com/workers/) to proxy an
incoming request and then transform cookies in the response. This first version
will:

- Remove any invalid `SameSite` attributes
- Add `Secure` to any cookie with `SameSite=None`
- Add `SameSite=Lax` by default

The `COOKIEMAP` can be used to apply a given `SameSite` value to the stated cookie.


```javascript
/*
 * Try https://samesite-proxy.rowan.workers.dev/proxied
 */

const COOKIEMAP = {
  ck06: 'strict',
  ck07: 'lax',
  ck08: 'none'
}

async function handleRequest(request) {
  const url = new URL(request.url);
  url.hostname = 'samesite-sandbox.glitch.me';
  const response = await fetch(url, request);

  const alteredHeaders = new Headers(response.headers);
  // todo fix regex to deal with commas in the Set-Cookie string
  const cookies  = alteredHeaders.get('set-cookie').match(/[^,].+?(?=,|$)/g);
  alteredHeaders.delete('set-cookie');

  for (const cookie of cookies) {
    const directives = cookie.split('; ');
    const cookieName = directives[0].match(/(\S+)(?=\=)/g);
    let isSecure = false;
    let secureIdx = null;
    let hasSameSite = false;
    let sameSiteIdx = null;

    directives.forEach((directive, index) => {
      const lcDirective = directive.toLowerCase();

      switch (lcDirective) {
        case 'secure':
          isSecure = true;
          secureIdx = index;
          break;
        case 'samesite=strict':
          hasSameSite = 'strict';
          sameSiteIdx = index;
          break;
        case 'samesite=lax':
          hasSameSite = 'lax';
          sameSiteIdx = index;
          break;
        case 'samesite=none':
          hasSameSite = 'none';
          sameSiteIdx = index;
          break;
      }

      // Remove any invalid SameSite values
      if (!hasSameSite && lcDirective.startsWith('samesite=')) {
        directives.splice(index, 1);
      }
    });

    if (COOKIEMAP[cookieName] && !hasSameSite) {
      directives.push('SameSite=' + COOKIEMAP[cookieName]);
      hasSameSite = COOKIEMAP[cookieName];
    }

    // SameSite=None must include Secure
    if (hasSameSite === 'none' && !isSecure) {
      directives.push('Secure');
    }

    // SameSite=Lax by default
    if (!hasSameSite) {
      directives.push('SameSite=Lax');
    }
    
    alteredHeaders.append('set-cookie', directives.join('; '));
  }

  const alteredResponse = new Response(response.body, {
    status: response.status,
    statusText: response.statusText,
    headers: alteredHeaders
  });

  return alteredResponse;
}

addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
});
```
