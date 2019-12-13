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

// Site to proxy
const TARGET = 'samesite-sandbox.glitch.me';

// Valid SameSite values
const STRICT = 'Strict';
const LAX = 'Lax';
const NONE = 'None';

// Cookie names and the corresponding SameSite value to enforce
const SAMESITE_MAP = {
  ck06: STRICT,
  ck07: LAX,
  ck08: NONE,
  ck09: NONE
};

// Strip any invalid SameSite attributes from cookies
const REMOVE_INVALID_SAMESITE = true;

// If no valid SameSite attribute is present, add this one
// false / null to disable this
const APPLY_DEFAULT_SAMESITE = LAX;

// Add missing Secure attribute to SameSite=None cookies
const APPLY_SECURE_SAMESITE_NONE = true;

// Mitigation strategy - sniff the user-agent and only add attribute for supported browsers
const UA_SNIFF = 'ua-sniff';
// Mitigation strategy - send two versions of the cookie, one with SameSite, one without
const DOUBLE_COOKIE = 'double-cookie';
// Prefix to use when doubling cookies
const LEGACY_PREFIX = 'legacy_';

// Cookie names and the corresponding mitigation strategy to apply
const MITIGATION_MAP = {
  ck08: UA_SNIFF,
  ck09: DOUBLE_COOKIE
};

APPLY_DEFAULT_MITIGATION = DOUBLE_COOKIE;

async function handleRequest(request) {
  const url = new URL(request.url);
  url.hostname = TARGET;
  const response = await fetch(url, request);

  const alteredHeaders = new Headers(response.headers);

  // Regex to split comma-separated Set-Cookie headers
  // Warning: it's a regex, who knows what's going on in there.
  const cookies = alteredHeaders.get('set-cookie').match(/[^,|\s].+?(?=, \S+=|$)/g);
  alteredHeaders.delete('set-cookie');

  // Will test the useragent for compatibility, but only need to do that once per request
  let isClientSameSiteNoneIncompatible = null;

  for (const cookie of cookies) {
    // Warning: splitting the directives assumes that ; isn't used in the cookie value
    const directives = cookie.split(/;\s+/);
    const cookieName = String(directives[0].match(/(\S+)(?=\=)/g));
    const cookieValue = directives[0].slice(cookieName.length + 1);

    let isSecure = false;
    let hasSameSite = null;
    let sameSiteIdx = null;

    if (
      isClientSameSiteNoneIncompatible === null
      && (MITIGATION_MAP[cookieName] === UA_SNIFF || APPLY_DEFAULT_MITIGATION === UA_SNIFF)
      && request.headers.get('user-agent')
    ) {
      isClientSameSiteNoneIncompatible = isSameSiteNoneIncompatible(request.headers.get('user-agent'));
    }

    directives.forEach((directive, index) => {
      const lcDirective = directive.toLowerCase();

      switch (lcDirective) {
        case 'secure':
          isSecure = true;
          break;
        case 'samesite=strict':
          hasSameSite = STRICT;
          sameSiteIdx = index;
          break;
        case 'samesite=lax':
          hasSameSite = LAX;
          sameSiteIdx = index;
          break;
        case 'samesite=none':
          hasSameSite = NONE;
          sameSiteIdx = index;
          break;
      }

      // Remove any invalid SameSite values
      if (REMOVE_INVALID_SAMESITE && !hasSameSite && lcDirective.startsWith('samesite=')) {
        directives.splice(index, 1);
      }
    });

    // Apply specified SameSite attribute values
    const shouldSkip = (
      isClientSameSiteNoneIncompatible
      && (
        (SAMESITE_MAP[cookieName] === NONE && MITIGATION_MAP[cookieName] === UA_SNIFF)
        || APPLY_DEFAULT_MITIGATION === UA_SNIFF
      )
    );

    if (SAMESITE_MAP[cookieName] && !hasSameSite && !shouldSkip) {
      directives.push('SameSite=' + SAMESITE_MAP[cookieName]);
      hasSameSite = SAMESITE_MAP[cookieName];
      sameSiteIdx = directives.length - 1;
    }

    // Apply a default SameSite value if configured
    const shouldSkipDefault = (isClientSameSiteNoneIncompatible && APPLY_DEFAULT_SAMESITE === NONE);

    if (APPLY_DEFAULT_SAMESITE && !hasSameSite && !shouldSkipDefault) {
      directives.push('SameSite=' + APPLY_DEFAULT_SAMESITE);
      sameSiteIdx = directives.length - 1;
    }

    // Apply double cookie mitigation for SameSite=None where specified
    if (
      (
        MITIGATION_MAP[cookieName] === DOUBLE_COOKIE
        || APPLY_DEFAULT_MITIGATION === DOUBLE_COOKIE
      )
      && hasSameSite === NONE
    ) {
      const doubledDirectives = directives.slice(1);
      doubledDirectives.unshift(LEGACY_PREFIX + cookieName + '=' + cookieValue);
      doubledDirectives.splice(sameSiteIdx, 1);
      alteredHeaders.append('set-cookie', doubledDirectives.join('; '));
    }

    // Apply SameSite=None must include Secure if configured
    if (APPLY_SECURE_SAMESITE_NONE && hasSameSite === NONE && !isSecure) {
      directives.push('Secure');
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

function coalesceLegacyCookies(request) {
  const cookiePairs = request.headers.get('cookie').split(/;\s+/);
  const cookies = {};
  const legacyCookies = {};

  cookiePairs.forEach((cookie, index) => {
    const equalsIdx = cookie.indexOf('=');
    let cookieName = cookie.slice(0, equalsIdx);
    const cookieValue = cookie.slice(equalsIdx + 1);

    if (cookie.startsWith(LEGACY_PREFIX)) {
      cookieName = cookieName.substr(LEGACY_PREFIX.length);
      legacyCookies[cookieName] = cookieValue;
    } else {
      cookies[cookieName] = cookieValue;
    }
  });

  const alteredCookies = { ...legacyCookies, ...cookies };
  const alteredCookiePairs = [];

  Object.entries(alteredCookies).forEach(([key, value]) => {
    alteredCookiePairs.push(key + '=' + value);
  });

  request = new Request(request);
  request.headers.set('cookie', alteredCookiePairs.join('; '));

  return request;
}

addEventListener('fetch', event => {
  const request = coalesceLegacyCookies(event.request);
  event.respondWith(handleRequest(request));
});

// Don’t send `SameSite=None` to known incompatible clients.
function shouldSendSameSiteNone(useragent) {
  return !isSameSiteNoneIncompatible(useragent);
}

// Classes of browsers known to be incompatible.
function isSameSiteNoneIncompatible(useragent) {
  return hasWebKitSameSiteBug(useragent) ||
    dropsUnrecognizedSameSiteCookies(useragent);
}

function hasWebKitSameSiteBug(useragent) {
  return isIosVersion(12, useragent) ||
    (isMacosxVersion(10, 14, useragent) &&
      (isSafari(useragent) || isMacEmbeddedBrowser(useragent)));
}

function dropsUnrecognizedSameSiteCookies(useragent) {
  if (isUcBrowser(useragent)) {
    return !isUcBrowserVersionAtLeast(12, 13, 2, useragent);
  }

  return isChromiumBased(useragent) &&
    isChromiumVersionAtLeast(51, useragent) &&
    !isChromiumVersionAtLeast(67, useragent);
}

// Regex parsing of User-Agent string. (See note above!)
function isIosVersion(major, useragent) {
  const regex = /\(iP.+; CPU .*OS (\d+)[_\d]*.*\) AppleWebKit\//g;
  // Extract digits from first capturing group.
  const match = useragent.match(regex);
  return match && match[0] == major;
}

function isMacosxVersion(major, minor, useragent) {
  const regex = /\(Macintosh;.*Mac OS X (\d+)_(\d+)[_\d]*.*\) AppleWebKit\//g;
  // Extract digits from first and second capturing groups.
  const match = useragent.match(regex);
  return match && (match[0] == major) &&
    (match[1] == minor);
}

function isSafari(useragent) {
  const safari_regex = /Version\/.* Safari\//g;
  return safari_regex.test(useragent) &&
    !isChromiumBased(useragent);
}

function isMacEmbeddedBrowser(useragent) {
  const regex = /^Mozilla\/[\.\d]+ \(Macintosh;.*Mac OS X [_\d]+\) AppleWebKit\/[\.\d]+ \(KHTML, like Gecko\)$/g;
  return regex.test(useragent);
}

function isChromiumBased(useragent) {
  const regex = /Chrom(e|ium)/g;
  return regex.test(useragent);
}

function isChromiumVersionAtLeast(major, useragent) {
  const regex = /Chrom[^ \/]+\/(\d+)[\.\d]* /g;
  // Extract digits from first capturing group.
  const version = useragent.match(regex)[0];
  return version >= major;
}

function isUcBrowser(useragent) {
  const regex = /UCBrowser\//g;
  return regex.test(useragent);
}

function isUcBrowserVersionAtLeast(major, minor, build, useragent) {
  const regex = /UCBrowser\/(\d+)\.(\d+)\.(\d+)[\.\d]* /g;
  // Extract digits from three capturing groups.
  const major_version = useragent.match(regex)[0];
  const minor_version = useragent.match(regex)[1];
  const build_version = useragent.match(regex)[2];

  if (major_version != major) {
    return major_version > major;
  }

  if (minor_version != minor) {
    return minor_version > minor;
  }

  return build_version >= build;
}
```
