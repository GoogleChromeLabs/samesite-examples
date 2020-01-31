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

# Elixir - Phoenix example for `SameSite=None; Secure`

Once [the fix to this issue](https://github.com/elixir-plug/plug/pull/861) is
released, you will be able to use
`:same_site`
like this:

```elixir
plug Plug.Session,
  store: :cookie,
  same_site: :none,
  secure: true,
  #...
```

While you're waiting for the release, you can still
[set the header](https://hexdocs.pm/plug/Plug.Conn.html#put_resp_cookie/4)
explicitly through `:extra` key to pass any string that is appended to the cookie:

```elixir
plug Plug.Session,
  store: :cookie,
  extra: "SameSite=None",
  secure: true,
  #...
```
