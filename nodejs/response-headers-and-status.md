# Response headers and status

### Response header
A **response header** is an HTTP header that can be used in an HTTP response and that doesn't relate to the content of the message. Response headers, like Age, Location or Server are used to give a more detailed context of the response.

<u>Standard headers</u>

| Headers | Descriptions | Example Value |
| - | - | - |
| Accept-Patch | Specifies which patch document formats this server supports | text/example; charset=utf-8 |
| Accept-Ranges | What partial content range types this server supports via byte serving | bytes |
| Age | The age the object has been in a proxy cache in seconds | 12 |
| Allow | Valid methods for a specified resource. To be used for a 405 Method not allowed | GET, HEAD |
| Alt-Svc | A server uses "Alt-Svc" header (meaning Alternative Services) to indicate that its resources can also be accessed at a different network location (host or port) or using a different protocol. When using HTTP/2, server should instead send an ALTSVC frame | http/1.1= "http2.example.com:8001"; ma=7200 |
| Cache-Control | If no-cache is used, the Cache-Control header can tell the browser to never use a cached version of a resource without first checking the ETag value. max-age is measured in seconds. The more restrictive no-store option tells the browser (and all the intermediary network devices) the not even store the resource in its cache | max-age=3600, no-cache, no-store, max-age=0, must-revalidate |
| Connection | Control options for the current connection and list of hop-by-hop response fields. Deprecated in HTTP/2 | close |
| Content-Disposition | An opportunity to raise a "File Download" dialogue box for a known MIME type with binary format or suggest a filename for dynamic content. Quotes are necessary with special characters | attachment; filename="file.txt” |
| Content-Encoding | The type of encoding used on the data | gzip |
| Content-Length | The length of the response body expressed in 8-bits bytes | 348 |
| Content-Range | Where in a full body message this partial message belongs | bytes 21010-47021/47022 | 
| Date | The date and time that the message was sent (in "HTTP-date" format as defined by RFC 7231) | Tue, 15 Nov 1994 08:12:31 GMT |
| Expires | Gives the date/time after which the response is considered state (in "HTTP-date" format as defined by RFC 7231) | Sat, 01 Dec 2018 16:00:00 GMT |
| Last-Modified | The last modified date for the requested object (in “HTTP-date” format as defined by RFC 7231) | Mon, 15 Nov 2017 12:00:00 GMT |
| Link | Used to express a typed relationship with another resource, where the relation type is defined by RFC 5988 | </feed>; rel="alternate” |
| Location | Used in redirection, or when a new resource has been created | /pub/WWW/People.html | 
| Proxy-Authenticate | Request authentication to access the proxy | Basic |
| Public-Key-Pins | HTTP Public Key Pinning, announces hash of website's authentic TLS certificate | | 
| Retry-After | If an entity is temporarily unavailable, this instructs the client to try again later. Value could be a specified period of time (in seconds) or a HTTP-date | Retry-After: 120; Retry-After: Fri, 07 Nov 2014 23:59:59 GMT | 
| Server | A name for the server | Apache/2.4.1 (Unix) | 
| Set-Cookie | An HTTP cookie | Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1 | 
| Via | Informs the client of proxies through which the response was sent | Via: 1.0 fred, 1.1 example.com (Apache/1.1) | 
| WWW-Authenticate | Indicates the authentication scheme that should be used to access the requested entity | WWW-Authenticate: Basic |

<u>CORS headers</u>
- Access-Control-Allow-Origin
- Access-Control-Allow-Credentials 
- Access-Control-Expose-Headers 
- Access-Control-Max-Age
- Access-Control-Allow-Methods
- Access-Control-Allow-Headers

<u>Non-standard headers:</u>
| Headers | Descriptions | Example values |
| - | - | - | 
| Content-Security-Policy | Helps to protect against XSS attacks. | |
| Refresh | Redirect to a URL after an arbitrary delay expressed in seconds | Refresh: 10;http://www.example.org/ | 
| X-Powered-By | Can be used by servers to send their name and version | X-Powered-By: Brain/0.6b | 
| X-Request-ID | Allows the server to pass a request ID that clients can send back to let the server correlate the request | | 
| X-UA-Compatible | Sets which version of Internet Explorer compatibility layer should be used. Only used if you need to support IE8 or IE9 |
| X-XSS-Protection | Now replaced by the Content-Security-Policy header, used in older browsers to stop pages load when an XSS attack is detected | 

### HTTP response status codes
HTTP response status codes indicate whether a specific HTTP request has been successfully completed. Responses are grouped in 5 classes:

| Classes | Descriptions |
| - | -  | 
| Informational responses (100-199) | It means the request has been received and the process is continuing |
| Successful responses (200-299) | It means the action was successfully received, understood, and accepted |
| Redirection messages (300-399) | It means further action must be taken in order to complete the request | 
| Client error responses (400-499) | It means the request contains incorrect syntax or cannot be fulfilled |
| Server error responses (500-599) | It means the server failed to fulfill an apparently valid request |

