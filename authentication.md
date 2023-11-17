# Authentication
Authentication is the process of identifying whether a client is eligible to access a resource. The HTTP protocol supports authentication as a means of negotiating access to a secure resource.

### Authentication schemes
The general HTTP authentication framework is the base for a number of authentication schemes. IANA maintains a list of authentication schemes, but there are other schemes offered by host services, such as Amazon AWS.

Some common authentication schemes include:
- **Basic**: See RFC 7617, base64-encoded credentials.
- **Bearer**: See RFC 6750, bearer tokens to access OAuth 2.0-protected resources.
- **Digest**: See RFC 7616, Firefox 93 and later support SHA-256 encryption. Previous versions only support MD5 hashing (not recommended).
- **HOBA**: See RFC 7486, HTTP Origin-Bound Authentication, digital-signature-based
- **Mutual**: RFC 8120
- **Negotiate** / NTLM: RFC 4599
- **VAPID**: RFC 8292
- **SCRAM**: RFC 7804
- **AWS4-HMAC-SHA256**: This scheme is used for AWS3 server authentication.

Schemes can differ in security strength and in their availability in client or server software.

### A problem of state
The HTTP protocol is stateless, this means a new request (e.g. GET /order/42 ) won't know anything about the previous one, so we need to reauthenticate for each new request.

<U>The better solution</u>

JSON Web Token (JWT) is an open standard (RFC 7519) that defines a way for transmitting information - like authentication and authorization facts - between two parties: an issuer and an audience. Communication is safe because each token issued is digitally signed, so the consumer can verify if the token is authentic or has been forged.

Each token is self-contained, this means it contains all information needed to allow or deny any given requests to an API.

<U>Anatomy of a JWT </u>
A JSON Web Token is essentially a long encoded text string. This string is composed of 3 smaller parts, separated by a dot sign. These parts are:
- the header;
- a payload or body;
- a signature;
Therefore, tokens will look like this: header.payload.signature

