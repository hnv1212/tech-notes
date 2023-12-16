# Request headers and methods

Common **HTTP Request Headers:**
| Headers | Descriptions | Example values |
| - | - | - |
| Accept | The media type/types acceptable | application/json |
| Accept-Charset | The charset acceptable | utf-8 |
| Accept-Encoding | List of acceptable encodings | gzip, deflate | 
| Accept-Language | List of acceptable languages | en-US |
| Accept-Datetime | Request a past version of the resource prior to the datetime passed | Thu, 31 May 2007 20:35:00 GMT |
| Access-Control-Request-Method | Used in a CORS request | GET |
| Access-Control-Request-Headers | Used in a CORS request | origin, x-requested-with, accept |
| Authorization | HTTP basic authentication credentials | Bearer 34i3j4iom2323== |
| Cache-Control | Set the caching rules | no-cache |
| Connection | Control options for the current connection. Accepts keep-alive and close. Deprecated in HTTP/2 | keep-alive |
| Content-Length | The length of the request body in bytes | 348 |
| Content-Type | The content type of the body of the request (used in POST and PUT requests) | application/x-www-form-urlencoded |
| Cookie | See more on Cookies [https://flaviocopes.com/cookies/] | name=value |
| Date | The date and time that the request was sent | Tue, 15 Nov 1994 08:12:31 GMT |
| Expect | It's typically used when sending a large request body. We expect the server to return back a 100 Continue HTTP status if it can handle the request, or 417 Expectation Failed if not | 100-continue |
| Forwarded | Disclose original information of a client connecting to a web server through an HTTP proxy. Used for testing purposes only, as it discloses privacy sensitive information | for=192.0.2.60; proto=http; by=203.0.113.43 |
| Host | The domain name of the server (used to determined the server with virtual hosting), and the TCP port number on which the server is listening. If the port is omitted, 80 is assumed. This is a mandatory HTTP request header | examplehost.com |
| If-Match | Given one (or more) ETags, the server should only send back the response if the current resource matches one of those ETags. Mainly used in PUT methods to update a resource only if it has not been modified since the user last updated it | "737060cd8c284d8582d” |
| If-Modified-Since | Allows to return a 304 Not Modified response header if the content is unchanged since that date | Sat, 29 Oct 1994 19:43:31 GMT |
| If-None-Match | Allows a 304 Not Modified response header to be returned if content is unchanged. Opposite of If-Match | "737060cd882f209582d” |
| Max-Forwards | Limit the number of times the message can be forwarded through proxies or gateways | 10 | 
| Origin | Send the current domain to perform a CORS request, used in an OPTIONS HTTP request (to ask the server for Access-Control- response headers) | http://mydomain.com |
| X-Requested-With | Identifies XHR requests | XMLHttpRequest | 

Common **HTTP Request methods**:
| HTTP Verb | Description |
| - | - |
| GET | The GET method requests a representation of the specified resource. Requests using GET should only retrieve data |
| HEAD | The HEAD method asks for a response identical to a GET request, but without the response body |
| POST | The POST method submits an entity to the specified resource, often causing a change in state or side effects on the server |
| PUT | The PUT method replaces all current representations of the target resource with the request payload | 
| DELETE | The DELETE method deletes the specified resource |
| CONNECT | The CONNECT method establishes a tunnel to the server identified by the target resource |
| OPTIONS | The OPTIONS method describes the communication options for the target resource |
| TRACE | The TRACE method performs a message loop-back test along the path to the targety resource |
| PATCH | The PATCH method applies partial modifications to a resource