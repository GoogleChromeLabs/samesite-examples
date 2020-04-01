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

# Python - Django example for `SameSite=None; Secure`

Once [the fix in this pull request](https://github.com/django/django/pull/11894) is
released (currently scheduled for [Django 3.1, stable in Aug 2020](https://code.djangoproject.com/wiki/Version3.1Roadmap)), you will be able to use:

 * [`CSRF_COOKIE_SAMESITE`](https://docs.djangoproject.com/en/3.0/ref/settings/#csrf-cookie-samesite)` = 'None'`
 * [`LANGUAGE_COOKIE_SAMESITE`](https://docs.djangoproject.com/en/3.0/ref/settings/#language-cookie-samesite)` = 'None'`
 * [`SESSION_COOKIE_SAMESITE`](https://docs.djangoproject.com/en/3.0/ref/settings/#session-cookie-samesite)`= 'None'`

This will also allow the `'None'` string in the [`HttpResponse.set_cookie()`](https://docs.djangoproject.com/en/3.0/ref/request-response/#django.http.HttpResponse.set_cookie) method.

```python
def home(request, template):
    response = render(request, template)  # django.http.HttpResponse
    response.set_cookie(key='same-site-cookie', value='foo', samesite='Lax')
    response.set_cookie(key='cross-site-cookie', value='bar', samesite='None', secure=True)
    return response
```

