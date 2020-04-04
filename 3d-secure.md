<!--
 Copyright 2020 Google Inc.

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

# Fixing issues with `SameSite` cookies and 3-D Secure checkout flows

3-D Secure is a verification service for card payments taken online. From the
point of view of a user or site, the flow generally looks like this:

- User enters their payment details on the site’s checkout;
- Site embeds or redirects to the appropriate page to verify that payment method
  (dependent on the user’s card, e.g. Visa, Amex, etc.);
- Third-party does its verification (via SMS or similar);
- Third-party returns the status to the main site, generally via a `POST`
  request.

The 3-D Secure interface may be displayed to the user in a number of different
ways: an iframe in the page, a pop-up, or a full page redirect. The common
pattern with all of these is a cross-site `POST` request initiated at the end of
the 3-D Secure verification back to the original site. This is what causes a
challenge with the recent updates to apply a `SameSite=Lax` by default behaviour
to cookies without a `SameSite` value. In other words, **cookies without a
`SameSite` attribute will not be included on the 3-D Secure `POST` callback**.

This means that final `POST` request may appear to your server as a new session,
meaning your site may attempt to set new cookies for the user or treat them as
if they were not logged in.

![Sequence diagram showing cookies being excluded on the final POST request](https://cdn.glitch.com/e304e580-9956-4cb6-9f62-9e89ad6cd85f%2Fsamesite-3d-secure-fail.png?v=1585762915129)

There are a number of options to fix this based on your site's archicture.

## Do not require cookies on the returning `POST` request

Generally, the cookies on your site for maintaining the user's session are only
intended for first-party use. You do not want them sent on cross-site requests
as this loses the security benefits of implementing `SameSite=Lax` by default,
e.g. it makes Cross-Site Request Forgery attacks easier.

Check if you are able to process the returning `POST` request without the need
for any of your session cookies. For example, if the 3-D Secure challenge was
included in an `iframe` then you may only need to display a status message
within the frame while sending a `postmessage()` call to your top-level window
to complete the transaction.

If the returning `POST` request is a top-level navigation / redirect and you
would like to display session-specific content to the user, then you may want to
consider the
[`POST`/Redirect/`GET` pattern](https://en.wikipedia.org/wiki/Post/Redirect/Get).
That is:

- Process the incoming `POST` request without cookies recording any results you
  need in the back-end
- Respond with a `303` redirect to an internal page for the result of the
  transaction, e.g. `/transaction/123/success` or similar.
- The subsequent `GET` request is considered a "safe" method and will include
  any `SameSite=Lax` cookies.

This means that the redirected request will include your sites first-party
cookies and can be used to display session-specific content.

## Create short-lived, cross-site cookies for the returning `POST` request

If you do need cookies on the returning `POST` request, they need to be marked
with the `SameSite=None; Secure` attributes. You should not make your default
session cookies available to third-parties, so instead consider creating a
short-lived cookie that stores a token that allows you to link the incoming
`POST` request to the associated user or transaction on your back-end.

This scenario can be implement as follows:

- Create a short-lived cookie with the `SameSite=None; Secure` attributes for
  retrieve session id after payment
- Check The HTTP referer on the incoming `POST` request to make sure the 
  request is come from trusted source 
- Use the short-lived cookie for set-up session id and retrieve the current
  session information
- The short-lived cookie is no longer needed, so destroy it
