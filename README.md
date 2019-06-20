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

# `SameSite` examples

This is a companion repo for the
["`SameSite` cookies explained"](https://web.dev/samesite-cookies-explained)
article on web.dev. This is your starting point for how cookies work, the
functionality of the `SameSite` attribute, and the changes in Chrome to apply a
`SameSite=Lax` policy by default while requiring the use of
`SameSite=None; Secure` for cookies in a third-party context.

This functionality is available now in
[Chrome 76](https://www.chromestatus.com/features/schedule) behind the
associated flags to let you test the effect on your site. The Chrome Status
entries provide the proposed releases for this behaviour to become the default.

## `SameSite=Lax` by default

- Flag: `chrome://flags/#same-site-by-default-cookies`
- Chrome Status entry:
  [Cookies with `SameSite` by default](https://www.chromestatus.com/feature/5088147346030592)

## Require `Secure` with `SameSite=None`

- Flag: `chrome://flags/#cookies-without-same-site-must-be-secure`
- Chrome Status entry:
  [Reject insecure `SameSite=None` cookies](https://chromestatus.com/feature/5633521622188032)

# Examples

In this repo you'll find examples on making use of `SameSite=None; Secure` in a
variety of languages, libraries, and frameworks. The `SameSite` attribute is
widely supported, but the addition of the explicit `None` value may require
updates or work-arounds.

If your specific platform isn't covered here, please raise an issue or a pull
request to include it.

- [JavaScript](javascript.md)
- [JavaScript - Node.js](javascript-nodejs.md)
- [PHP](php.md)
- [Python](python.md)

## Contributing

Issues and pull requests are always welcome. For details, see
[CONTRIBUTING](CONTRIBUTING.md)

This is not an officially supported Google product.
