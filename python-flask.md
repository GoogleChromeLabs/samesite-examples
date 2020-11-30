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

# Python - Flask example for `SameSite=None; Secure`

Once [the fix to this issue](https://github.com/pallets/werkzeug/issues/1549) is
released, you will be able to use
[`set_cookie()`](http://flask.pocoo.org/docs/1.0/api/#flask.Response.set_cookie)
like this:

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.route('/')
def hello_world():
    resp = make_response('Hello, World!')
    # Set a same-site cookie for first-party contexts
    resp.set_cookie('cookie1', 'value1', samesite='Lax')
    # Set a cross-site cookie for third-party contexts
    resp.set_cookie('cookie2', 'value2', samesite='None', secure=True)
    return resp
```

While you're waiting for the release, you can still
[set the header](https://werkzeug.palletsprojects.com/en/0.15.x/datastructures/#werkzeug.datastructures.Headers.add)
explicitly:

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.route('/')
def hello_world():
    resp = make_response('Hello, World!')
    # Set a same-site cookie for first-party contexts
    resp.set_cookie('cookie1', 'value1', samesite='Lax')
    # Ensure you use "add" to not overwrite existing cookie headers
    # Set a cross-site cookie for third-party contexts
    resp.headers.add('Set-Cookie','cookie2=value2; SameSite=None; Secure')
    return resp
```
