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

# 🍪 `SameSite` examples

---

## 💡 Upcoming deprecation of third-party cookies

Now that support for `SameSite=None` has been deployed across the web, the next step is restricting third-party cookies by default. If you added `SameSite=None` to any of your cookies, then you will now need to take further action to migrate or maintain that functionality.

**See the [Third-party cookie deprecation section in the Privacy Sandbox developer documentation](https://developers.google.com/privacy-sandbox/3pcd).**

---

This is a companion repo for the
["`SameSite` cookies explained"](https://web.dev/samesite-cookies-explained)
article on web.dev. This is your starting point for how cookies work, the
functionality of the `SameSite` attribute, and the changes in Chrome to apply a
`SameSite=Lax` policy by default while requiring the use of
`SameSite=None; Secure` for cookies in a third-party context.

This functionality is available now in
[Chrome 76](https://www.chromestatus.com/features/schedule) behind the
associated flags to let you test the effect on your site. This is intended to
become default behaviour as of Chrome 80.

## `SameSite=Lax` by default

- Flag: `chrome://flags/#same-site-by-default-cookies`
- Chrome Status entry:
  [Cookies with `SameSite` by default](https://www.chromestatus.com/feature/5088147346030592)

Turn this flag on to have Chrome apply the equivalent of `SameSite=Lax` to
cookies without a `SameSite` attribute specified.

## Require `Secure` with `SameSite=None`

- Flag: `chrome://flags/#cookies-without-same-site-must-be-secure`
- Chrome Status entry:
  [Reject insecure `SameSite=None` cookies](https://chromestatus.com/feature/5633521622188032)

Turn on this flag along with the previous flag to have Chrome enforce the need
for any `SameSite=None` cookie to also specify the `Secure` attribute.

## See affected cookies

- Flag `chrome://flags/#cookie-deprecation-messages`

This will add console warning messages for _every single cookie_ potentially
affected by this change.

**⚠️ WARNING:** You will see a lot of messages! Seriously, a lot of messages.

Since the vast majority of cookies do not have any `SameSite` attribute set that
means they are all sent in a cross-site context, regardless of whether or not
the intent is to use them.

As you add the correct `SameSite` and `Secure` values to your cookies, you will
be able to use the console warnings to test for any you have missed. Try this
without the previous flags enabled.

# ⚙️ Examples

In this repo you'll find examples on making use of `SameSite=None; Secure` in a
variety of languages, libraries, and frameworks. The `SameSite` attribute is
widely supported, but the addition of the explicit `None` value may require
updates or work-arounds.

**🚧 NOTE:** To test the `None` value is set you need to test in a browser that
parses this addition, e.g. Chrome 76 or above. The changes _should_ be backwards
compatible, but those browsers should ignore the `None` value so you will not
see it in any cookie view.

If your specific platform isn't covered here, please raise an issue or a pull
request to include it.

- [ASP.NET](aspnet.md)
- [Elixir - Phoenix](elixir-phoenix.md)
- [JavaScript](javascript.md)
- [JavaScript - Node.js](javascript-nodejs.md)
- [PHP](php.md)
- [Python](python.md)
- [Python - Flask](python-flask.md)
- [Shopify](https://help.shopify.com/en/api/guides/samesite-cookies)🌐↗️

# 📋 Use cases and common scenarios

- [Fixing issues with `SameSite` cookies and 3-D Secure checkout flows](3d-secure.md)

# 🙋 Questions

You can
[raise an issue in this repo](https://github.com/GoogleChromeLabs/samesite-examples/issues)
if there is specific behaviour you would like to see documented or something
that's not clear in the current examples.

You can also use the
[`samesite` tag on StackOverflow](https://stackoverflow.com/questions/tagged/samesite)
which we will monitor on a regular basis. As the discussion evolves there, we'll
also add a Frequently Asked Questions section to this repo for easy reference.

## 💻 Contributing

Issues and pull requests are always welcome. For details, see
[CONTRIBUTING](CONTRIBUTING.md)

This is not an officially supported Google product.
