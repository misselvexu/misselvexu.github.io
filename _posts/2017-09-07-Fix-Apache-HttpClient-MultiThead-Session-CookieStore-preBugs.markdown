---
layout:     post
title:      "[Fix] Apache HttpClient Multi-Threads Issues"
subtitle:   "small issues"
date:       2017-09-07 01:59:59
author:     "Elve Xu"
header-img: "img/in-post/cookie-bg.jpg"
catalog: true
tags:
    - Apache HttpClient
    - Cookie
    - Multi-Threads
---

> Code is Not Only Code .

### From Apache Document
#### What is Cookies?
> Cookies are the preferred way for servers to track sessions. The server supplies a piece of data, called a cookie, in response to a request. The server expects the client to send that piece of data in a header field with each following request of the same session. The cookie is different for each session, so the server can identify to which session a request belongs by looking at the cookie. If the cookie is missing from a request, the server will not respond as expected.

#### Cookie Manager [Cookie 管理]
> HttpClient automatically parses cookies sent in responses and puts them into a cookie store. HttpClient uses a configurable cookie policy to decide whether a cookie being sent from a server is correct. The default policy complies strictly with RFC 2109, but many servers do not. Play around with the cookie policies until the cookie is accepted and put into the cookie store.

> 大意: HttpClient会自动解析响应中发送的Cookie，并将其放入Cookie存储区。 HttpClient使用可配置的Cookie策略来确定从服务器发送的Cookie是否正确。
> 默认策略严格遵守RFC:2109，很多服务场景不适用。
> 反复重试Cookie政策，直到Cookie被接受并放到CookieStore。

### Issues-Scene
> Simulate Login into Apple-Development-Website with Apache HttpClient 4.x ,Multi-Threads use different account login together, Because Apple Server determines the login status(session) based on the Cookie , And then found user-information dislocation.
> According to Apache HttpClient document & source code 
> 
> Code snippet :

```
    @ThreadSafe
    public class BasicCookieStore implements CookieStore, Serializable {
    
        // all session cookie will store here
        // different website's cookie has no problem
        @GuardedBy("this")
        private final TreeSet<Cookie> cookies;
    
        public BasicCookieStore() {
            super();
            this.cookies = new TreeSet<Cookie>(new CookieIdentityComparator());
        }
        
        ....
        ....
    }
    
```

> Fix Code snippet

```
    // use threadlocal to isolate cookie
    private static final ThreadLocal<TreeSet<Cookie>> cookieCached = new ThreadLocal<TreeSet<Cookie>>() {
        @Override
        protected TreeSet<Cookie> initialValue () {
            return new TreeSet<Cookie>(new CookieIdentityComparator());
        }
    };
```

> It's working!



### Additional `CookieSpecs`
- netscape

> The Netscape cookie draft compliant policy

- standard

> The RFC 6265 compliant policy (interoprability profile)

- standard-strict

> The RFC 6265 compliant policy (strict profile).

- default

> The default policy. This policy provides a higher degree of compatibility with common cookie management of popular HTTP agents for non-standard (Netscape style) cookies.

- ignoreCookies

> The policy that ignores cookies.


### Links
- [RFC 6265](https://tools.ietf.org/html/rfc6265)
- [Client HTTP Programming Primer](https://hc.apache.org/httpcomponents-client-ga/primer.html)
- [ThreadLocalCookieStore.java]()

