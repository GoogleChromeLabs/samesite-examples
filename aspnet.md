# ASP.NET and ASP.NET Core

See the annoucements here: <br>
https://devblogs.microsoft.com/aspnet/upcoming-samesite-cookie-changes-in-asp-net-and-asp-net-core/ <br>
https://github.com/aspnet/Announcements/issues/390 <br>

Affected libraries like OpenIdConnect and WsFederation will automatically set SameSite=None once the patches are installed. Beware that this is not compatible with many browsers and user-agent sniffing will be required. Examples are given in the announcements.

## ASP.NET 4.7.2+
After installing the patch announced above, SameSite=None can be sent on a cookie using the following:
```
Response.Cookies.Add(new HttpCookie("key", "value")
{
    SameSite = SameSiteMode.None,
    Secure = true,
});
```


## ASP.NET Core 2.1+
After installing the patches announced above, SameSite=None can be sent on a cookie using the following:
```
Response.Cookies.Append("Key", "Value", new CookieOptions()
{
     SameSite = SameSiteMode.None,
     Secure = true,
});
```

