## Tomcat Version:- Apache Tomcat/9.0.31
### To add samesite=none to cookie 
* Open context.xml
```
inside the <context> tag add
<CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor" sameSiteCookies="none" />
```
*Example
```
<context>
    <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor" sameSiteCookies="none" />
</context>
```
### To add secure property to cookie

* Open server.xml

```
Inside the <session-config> tag add:-
<cookie-config>
    <secure>true</secure>
</cookie-config>
```

* Example
```
<session-config>
    <cookie-config>
    	<secure>true</secure>
	</cookie-config>
</session-config>
```

#### Save the changes & restart the server
