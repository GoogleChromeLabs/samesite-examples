# ASP.NET and ASP.NET Core

See the announcements here: <br>
https://devblogs.microsoft.com/aspnet/upcoming-samesite-cookie-changes-in-asp-net-and-asp-net-core/ <br>
https://github.com/aspnet/Announcements/issues/390 <br>

Affected libraries like OpenIdConnect and WsFederation will automatically set `SameSite=None` once the patches are installed. Beware that this is not compatible with many browsers and user-agent sniffing will be required. Examples are given in the announcements.

## ASP.NET 4.7.2+
After installing the patch announced above, `SameSite=None; Secure` can be sent on a cookie using the following:
```
Response.Cookies.Add(new HttpCookie("key", "value")
{
    SameSite = SameSiteMode.None,
    Secure = true,
});
```


## ASP.NET Core 2.1+
After installing the patches announced above, `SameSite=None; Secure` can be sent on a cookie using the following:
```
Response.Cookies.Append("Key", "Value", new CookieOptions()
{
     SameSite = SameSiteMode.None,
     Secure = true,
});
```

## < ASP.NET 4.7.2
Cookie needs to be overwritten with the appropriate attributes set in the Global.asax:
```
void PreSendRequestHeaders(object sender, EventArgs e)
{
    HttpApplication app = sender as HttpApplication;
    if (app != null && app.Context.SkipAuthorization && app.Request.Cookies["ASP.NET_SessionId"] != null)
        app.Response.AddHeader("Set-Cookie", "ASP.NET_SessionId=" + app.Request.Cookies["ASP.NET_SessionId"].Value + ";HttpOnly;SameSite=None;Secure");
}
```
