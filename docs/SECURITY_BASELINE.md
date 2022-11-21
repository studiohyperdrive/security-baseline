# SHD Security Baseline

This “Security Baseline” aims to collect a set of security guidelines and best practices to ensure a baseline level of security in the products we release. These guidelines are based on community and industry standards. They describe a set of rules without imposing technical choice or limitation.

Deviating from this baseline is allowed, if valid arguments can be provided, proving the need to extend, limit or otherwise change the set of rules described in this document. An addendum should be added in the project documentation to ensure the reasoning is known, now and in the future.

## Table of Contents

<details>
    <summary>Toggle</summary>

* [Security Baseline](#technical-specifications)
    * [Transport layer security (TLS/SSL)](#transport-layer-security-tlsssl)
        * [HTTPS](#https)
        * [Strict Transport Security](#strict-transport-security)
        * [Redirections](#redirections)
    * [HTTP Headers](#http-headers)
        * [Content Security Policy](#content-security-policy)
            * [Implementation Notes](#implementation-notes)
            * [Example](#example)
        * [Cross-origin Resource Sharing](#cross-origin-resource-sharing)
        * [Content Type Options](#content-type-options)
        * [Frame Options](#frame-options)
        * [XSS Protection](#xss-protection)
        * [XST Protection](#xst-protection)
        * [Cache Control](#cache-control)
    * [Resource loading](#resource-loading)
        * [CSRF Prevention](#csrf-prevention)
        * [Referrer policy](#referrer-policy)
        * [Subresource Integrity](#subresource-integrity)
    * [Sensitive information](#sensitive-information)
        * [User information](#user-information)
        * [Server information](#server-information)
    * [Cookies](#cookies)
    * [Robots.txt](#robotstxt)
* [Authentication](./AUTHENTICATION.md)
* [Mobile App Development](./MOBILE%20DEVELOPMENT.md)
</details>

---

# Technical specifications

## Transport layer security (TLS/SSL)

Transport Layer Security provides assurances about the confidentiality, authentication, and integrity of all communications both inside and outside the network. To protect our users and networked systems, the support and use of encrypted communications using TLS is mandatory for all systems.

Various security issues were detected and documented through the years. It is recommended to run TLS 1.2 or 1.3 and fully disable SSLv2, SSLv3, TLS 1.0 and TLS 1.1 that have protocol weaknesses. TSL 1.3 can be used [if supported by the targeted platforms](https://caniuse.com/?search=TLS%201.3), and should be preferred if all target platforms [support this version](https://wiki.mozilla.org/Security/Server\_Side\_TLS#Modern\_compatibility).

If security measures are properly implemented there is no additional security risk in using v1.2.

To verify supported TLS versions and capabilities, the [SSL Server Test by Qualys](https://www.ssllabs.com/ssltest/index.html) can be used.

For various configuration options (e.g. Apache, Nginx), the [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) can be used.

### HTTPS

HTTPS (HyperText Transfer Protocol Secure) is an encrypted version of the HTTP protocol. It uses SSL or TLS to encrypt all communication between a client and a server. This secure connection allows clients to safely exchange sensitive data with a server, such as when performing banking activities or online shopping.

Communication over HTTP (or cleartext traffic) should be avoided and disabled, HTTPS should always be preferred.

### Strict Transport Security

[HTTP Strict Transport Security (HSTS)](https://cheatsheetseries.owasp.org/cheatsheets/HTTP\_Strict\_Transport\_Security\_Cheat\_Sheet.html) is an HTTP header that notifies user agents to only connect to a given site over HTTPS, even if the scheme chosen was HTTP. Browsers that have had HSTS set for a given site will transparently upgrade all requests to HTTPS. HSTS also tells the browser to treat TLS and certificate-related errors more strictly by disabling the ability for users to bypass the error page.

The header consists of one mandatory parameter (max-age) and two optional parameters (includeSubDomains and preload), separated by semicolons:

**max-age**\
Indicates how long user agents will redirect to HTTPS, in seconds.\
Must be set to a minimum of six months (15768000), but longer periods such as two years (63072000) are recommended. Note that once this value is set, the site must continue to support HTTPS until the expiry time has been reached.

**includeSubDomains**\
Notifies the browser that all subdomains of the current origin should also be upgraded via HSTS. For example, setting “includeSubDomains” on “domain.example.com” will also set it on “host1.domain.example.com” and “host2.domain.example.com”. Extreme care is needed when setting the “includeSubDomains” flag, as it could disable sites on subdomains that don’t yet have HTTPS enabled.

**preload**\
Allows the website to be included in the [HSTS preload list](https://hstspreload.org/), upon submission. As a result, web browsers will do HTTPS upgrades to the site without ever having to receive the initial HSTS header. This prevents downgrade attacks upon first use and is recommended for all high risk websites. Note that being included in the HSTS preload list requires that includeSubDomains also be set.

### Redirections

Websites may continue to listen on port 80 (HTTP) so that users do not get connection errors when typing a URL into their address bar, as browsers currently connect via HTTP for their initial request.

Sites that listen on port 80 should only redirect to the same resource on HTTPS. Once the redirection has occurred, [HSTS ](https://infosec.mozilla.org/guidelines/web\_security#http-strict-transport-security)should ensure that all future attempts to go to the site via HTTP are instead sent directly to the secure site. APIs or websites not intended for public consumption should disable the use of HTTP entirely.&#x20;

Redirections should be done with the 301 redirects, unless they redirect to a different path, in which case they may be done with 302 redirections. Sites should avoid redirections from HTTP to HTTPS on a different host, as this prevents HSTS from being set.



## HTTP Headers

Properly configuring HTTP headers can go a long way to securing your client-to-server traffic.

You can use the [Security Headers project](https://securityheaders.com/) or the [validator provider by OWASP](https://github.com/oshp/oshp-validator) to validate traffic from and to your domain.

### Content Security Policy

[Content Security Policy (CSP)](https://infosec.mozilla.org/guidelines/web\_security#content-security-policy) is an HTTP header that allows site operators fine-grained control over where resources on their site can be loaded from. The use of this header is the best method to prevent cross-site scripting (XSS) vulnerabilities. Due to the difficulty in retrofitting CSP into existing websites, CSP is mandatory for all new websites and is strongly recommended for all existing high-risk sites.

The primary benefit of CSP comes from disabling the use of unsafe inline JavaScript. Inline JavaScript – either reflected or stored – means that improperly escaped user-inputs can generate code that is interpreted by the web browser as JavaScript. By using CSP to disable inline JavaScript, you can effectively eliminate almost all XSS attacks against your site.

Note that disabling inline JavaScript means that all JavaScript must be loaded from src tags . Event handlers such as onclick used directly on a tag will fail to work, as will JavaScript inside tags but not loaded via src. Furthermore, inline stylesheets using either tags or the style attribute will also fail to load. As such, care must be taken when designing sites so that CSP becomes easier to implement.

#### Implementation Notes

* It is recommended to start with a reasonably locked down policy such as:\
  `default-src 'none'; img-src 'self'; script-src 'self'; style-src 'self’` \
  and then add in sources as revealed during testing.
* In lieu of the preferred HTTP header, pages can instead include a tag:\
  `<meta http-equiv="Content-Security-Policy" content="…">`\
  If they do, it should be the first tag that appears inside the `<head>`.
* Care needs to be taken with data: URIs, as these are unsafe inside `script-src` and `object-src` (or inherited from `default-src`).
* Similarly, the use of `script-src 'self'` can be unsafe for sites with JSONP endpoints. These sites should use a `script-src` that includes the path to their JavaScript source folder(s).
* Unless sites need the ability to execute plugins such as Flash or Silverlight, they should disable their execution with `object-src 'none'`.
* Sites should ideally use the `report-uri` directive, which POSTs JSON reports about CSP violations that do occur. This allows CSP violations to be caught and repaired quickly.
* Prior to implementation, it is recommended to use the `Content-Security-Policy-Report-Only` HTTP header, to see if any violations would have occurred with that policy.

#### Example

<!-- {% code overflow="wrap" %} -->
```
default-src 'self' https: data: 'unsafe-inline' 'unsafe-eval'; connect-src 'self' https: *.custom-domain.com https://custom-url.com; image-src 'self' *.googleusercontent.com
```
<!-- {% endcode %} -->

<!-- {% hint style="info" %} -->
>☝️ You can use the [CSP evaluator tool from Google](https://csp-evaluator.withgoogle.com/) to validate your CSP header configuration.
<!-- {% endhint %} -->

### Cross-origin Resource Sharing

[Access-Control-Allow-Origin](https://infosec.mozilla.org/guidelines/web\_security#cross-origin-resource-sharing) is an HTTP header that defines which foreign origins are allowed to access the content of pages on your domain via scripts using methods such as XMLHttpRequest. `crossdomain.xml` and `clientaccesspolicy.xml` provide similar functionality, but for Flash and Silverlight-based applications, respectively.

These should not be present unless specifically needed. Use cases include content delivery networks (CDNs) that provide hosting for JavaScript/CSS libraries and public API endpoints. If present, they should be locked down to as few origins and resources as is needed for proper function.

For example, if your server provides both a website and an API intended for XMLHttpRequest access on a remote website, only the API resources should return the Access-Control-Allow-Origin header. Failure to do so will allow foreign origins to read the contents of any page on your origin.

### Content Type Options

[X-Content-Type-Options](https://infosec.mozilla.org/guidelines/web\_security#x-content-type-options) is a header supported by Internet Explorer, Chrome and Firefox 50+ that tells it not to load scripts and stylesheets unless the server indicates the correct MIME type.

Without this header, these browsers can incorrectly detect files as scripts and stylesheets, leading to XSS attacks.

As such, all sites must set the X-Content-Type-Options header and the appropriate MIME types for files that they serve.

### Frame Options

[X-Frame-Options](https://infosec.mozilla.org/guidelines/web\_security#x-frame-options) is an HTTP header that allows sites control over how your site may be framed within an iframe. Clickjacking is a practical attack that allows malicious sites to trick users into clicking links on your site even though they may appear to not be on your site at all. As such, the use of the X-Frame-Options header is mandatory for all new websites, and all existing websites are expected to add support for X-Frame-Options as soon as possible.

Note that X-Frame-Options has been superseded by the Content Security Policy’s **frame-ancestors directive**, which allows considerably more granular control over the origins allowed to frame a site.

As frame-ancestors are not yet supported in IE11 and older, Edge, Safari 9.1 (desktop), and Safari 9.2 (iOS), it is recommended that sites employ X-Frame-Options **in addition to using CSP**.

Sites that require the ability to be iframed must use either Content Security Policy and/or employ JavaScript defenses to prevent clickjacking from malicious origins.

### XSS Protection

[X-XSS-Protection](https://infosec.mozilla.org/guidelines/web\_security#x-xss-protection) is a feature of Internet Explorer and Chrome that stops pages from loading when they detect reflected cross-site scripting (XSS) attacks. Although these protections are largely unnecessary in modern browsers when sites implement a strong Content Security Policy that disables the use of inline JavaScript (`unsafe-inline`), they can still provide protections for users of older web browsers that don’t yet support CSP.

New websites should use the CSP header, the XSS protection should only be added if older platforms need to be supported.

This header is unnecessary for APIs, which should instead simply return a restrictive Content Security Policy header.

### XST Protection

A [Cross-Site Tracing (XST)](https://owasp.org/www-community/attacks/Cross\_Site\_Tracing) attack involves the use of Cross-site Scripting (XSS) and the `TRACE` or `TRACK` HTTP methods.

According to RFC 2616, “`TRACE` allows the client to see what is being received at the other end of the request chain and use that data for testing or diagnostic information.”, the `TRACK` method works in the same way but is specific to Microsoft’s IIS web server.

XST could be used as a method to steal user’s cookies via Cross-site Scripting (XSS) even if the cookie has the “HttpOnly” flag set or exposes the user’s Authorization header.

The TRACE method, while apparently harmless, can be successfully leveraged in some scenarios to steal legitimate users’ credentials. One of the most recurring attack patterns in Cross Site Scripting is to access the `document.cookie` object and send it to a web server controlled by the attacker so that they can hijack the victim’s session.

Tagging a cookie as HttpOnly forbids JavaScript to access it, protecting it from being sent to a third party. However, the `TRACE` method can be used to bypass this protection and access the cookie even in this scenario.

Modern browsers now prevent `TRACE` requests being made via JavaScript, however, other ways of sending `TRACE` requests with browsers have been discovered, such as using Java.

To prevent the `TRACE` requests from being abused:

* omit `TRACK` and `TRACE` as allowed Http methods in the application code (e.g. express / NestJS setup)
* disable TRACE requests in in Apache configuration (TraceEnable off)
* add an exception in nginx config:&#x20;

```nginx
if ($request_method ~ ^(PATCH|TRACE)$) {
  return 405;
}
```

### Cache Control

The [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) HTTP header holds directives (instructions) for caching in both requests and responses. A given directive in a request does not mean the same directive should be in the response.

The Cache-Control header is often set to `no-cache`, which implies the requests may be cached, but should always be validated against the origin server. If no additional validation is implemented the request can be cached anyway.

Prefer using `no-store` if you want to prevent the request from being cached.



## Resource loading

All resources — whether on the same origin or not — should be loaded over secure channels. Secure (HTTPS) websites that attempt to load active resources such as JavaScript insecurely will be blocked by browsers. As a result, users will experience degraded UIs and “mixed content” warnings. Attempts to load passive content (such as images) insecurely, although less risky, will still lead to degraded UIs and can allow active attackers to deface websites or phish users.

Despite the fact that modern browsers make it evident that websites are loading resources insecurely, these errors still occur with significant frequency. To prevent this from occurring, developers should verify that all resources are loaded securely prior to deployment.

### CSRF Prevention

[Cross-site request forgeries](https://infosec.mozilla.org/guidelines/web\_security#csrf-prevention) are a class of attacks where unauthorized commands are transmitted to a website from a trusted user. Because they inherit the users cookies (and hence session information), they appear to be validly issued commands.

A CSRF attack might look like this:

```html
<!-- Attempt to delete a user's account -->
<img src="https://accounts.mozilla.org/management/delete?confirm=true">
```

When a user visits a page with that HTML fragment, the browser will attempt to make a GET request to that URL. If the user is logged in, the browser will provide their session cookies and the account deletion attempt will be successful.

While there are a variety of mitigation strategies such as Origin/Referrer checking and challenge-response systems (such as CAPTCHA), the most common and transparent method of CSRF mitigation is through the use of anti-CSRF tokens. Anti-CSRF tokens prevent CSRF attacks by requiring the existence of a secret, unique, and unpredictable token on all destructive changes.&#x20;

These tokens can be set for an entire user session, rotated on a regular basis, or be created uniquely for each request. Although `SameSite` cookies are the best defense against CSRF attacks, they are not yet fully supported in all browsers and should be used in conjunction with other anti-CSRF defenses.

### Referrer policy

When a user navigates to a site via a hyperlink or a website loads an external resource, browsers inform the destination site of the origin of the requests through the use of the [HTTP Referer](https://infosec.mozilla.org/guidelines/web\_security#referrer-policy) (sic) header. Although this can be useful for a variety of purposes, it can also place the privacy of users at risk. HTTP Referrer Policy allows sites to have fine-grained control over how and when browsers transmit the HTTP Referer header.

In normal operation, if a page at `https://example.com/page.html` contains:

```html
<img src="https://not.example.com/image.jpg">
```

then the browser will send a request like this:

```
GET /image.jpg HTTP/1.1
Host: not.example.com
Referrer: https://example.com/page.html
```

In addition to the privacy risks that this entails, the browser may also transmit internal-use-only URLs that it may not have intended to reveal. If you as the site operator want to limit the exposure of this information, you can use HTTP Referrer Policy to either eliminate the Referer header or reduce the amount of information that it contains.

### Subresource Integrity

[Subresource integrity](https://infosec.mozilla.org/guidelines/web\_security#subresource-integrity) is a recent W3C standard that protects against attackers modifying the contents of JavaScript libraries hosted on content delivery networks (CDNs) in order to create vulnerabilities in all websites that make use of that hosted library.

For example, JavaScript code on jquery.org that is loaded from example.com has access to the entire contents of everything on “example.com”. If this resource was successfully attacked, it could modify download links, deface the site, steal credentials, cause denial-of-service attacks, and more.

Subresource integrity locks an external JavaScript resource to its known contents at a specific point in time. If the file is modified at any point thereafter, supporting web browsers will refuse to load it. As such, the use of subresource integrity is mandatory for all external JavaScript resources loaded from sources not hosted on Mozilla-controlled systems.

Note that CDNs must support the Cross Origin Resource Sharing (CORS) standard by setting the `Access-Control-Allow-Origin` header.



## Sensitive information

### Storing credentials

Sensitive information, such as user or server credentials, should always be stored securely in a dedicated Key Management System (KMS). Ensure to use a trusted KMS provider, such as:

* [Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/)
* [AWS KMS](https://aws.amazon.com/kms/)
* [GCP Cloud KMS](https://cloud.google.com/kms/docs/resource-hierarchy#key_rings)
* [Hashicorp Vault](https://www.vaultproject.io/)

Avoid storing user credentials with the application data, rather preferring a third-party solution that is focused on user identity and access management.

Credentials should never be stored in the codebase, appear in logs or made accessible in any other way that is not secured by a trusted provider.


### User information

Prevent storing and sending sensitive information as plain text, [even on a secure connection](https://owasp.org/www-project-top-ten/2017/A3\_2017-Sensitive\_Data\_Exposure.html). Identity information should always be transferred in an encrypted format (e.g. as part of the JWT token) and should never be cacheable.

When working with sessions, store the session information in a secure `httpOnly` [cookie](cookies.md). Make sure to (in)validate existing sessions or JWT tokens server side as well to avoid [unwanted access to protected resources](https://owasp.org/www-project-top-ten/2017/A5\_2017-Broken\_Access\_Control.html).

Make sure no stack traces or other information is leaked on server outages or other errors. Likewise, disable all introspection or monitoring tools on non-secured URLs.

### Server information

Disable any headers that expose information about used infrastructure & software.

* Nginx: set `server_tokens` to `off`
* Express.js: add [helmet](https://helmetjs.github.io/) (or implement similar behavior) to hide `X-Powered-By` headers and configure a decent set of default http headers

### Robots.txt

`robots.txt` is a text file placed within the root directory of a site that tells robots (such as indexers employed by search engines) how to behave, by instructing them not to crawl certain paths on the website. This is particularly useful for reducing load on your website through disabling the crawling of automatically generated content. It can also be helpful for preventing the pollution of search results, for resources that don’t benefit from being searchable.

Sites may optionally use `robots.txt`, but should only use it for these purposes. It should not be used as a way to prevent the disclosure of private information or to hide portions of a website. Although this does prevent these sites from appearing in search engines, it does not prevent its discovery from attackers, as robots.txt is frequently used for reconnaissance.


## Cookies

All [cookies](https://infosec.mozilla.org/guidelines/web\_security#cookies) should be created such that their access is as limited as possible. This can help minimize damage from cross-site scripting (see [XSS](http-headers/xss-protection.md) and [XST](http-headers/xst-protection.md) protection) vulnerabilities, as these cookies often contain session identifiers or other sensitive information.

* **Name**: Cookie names may be prepended with either `__Secure-` or `__Host-` to prevent cookies from being overwritten by insecure sources
  * Use `__Host-` for all cookies needed only on a specific domain (no subdomains) where Path is set to /
  * Use `__Secure-` for all other cookies sent from secure origins (such as HTTPS)
* **Secure**: All cookies must be set with the Secure flag, indicating that they should only be sent over HTTPS
* **HttpOnly**: Cookies that don’t require access from JavaScript should be set with the HttpOnly flag
* **Expiration**: Cookies should expire as soon as is necessary: session identifiers in particular should expire quickly
  * `Expires`: Sets an absolute expiration date for a given cookie
  * `Max-Age`: Sets a relative expiration date for a given cookie (not supported by IE <8)
* **Domain**: Cookies should only be set with this if they need to be accessible on other domains, and should be set to the most restrictive domain possible
* **Path**: Cookies should be set to the most restrictive path possible, but for most applications this will be set to the root directory
* **SameSite**: Forbid sending the cookie via cross-origin requests (such as from  tags, etc.), as a strong [anti-CSRF measure](https://infosec.mozilla.org/guidelines/web\_security#csrf-prevention)
  * `SameSite=Strict`: Only send the cookie when site is directly navigated to
  * `SameSite=Lax`: Send the cookie when navigating to your site from another site
