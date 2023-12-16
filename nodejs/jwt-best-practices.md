## Introduction
JWTs can be used for many different purposes. One of them is to transfer authentication and authorization claims between parties. For example, users who have authenticated with an identity provider may hold in their power a set of signed JWT claims that can be used by an application to confirm their identity.

In the identity space, JWTs are required by the OpenID Connect standard, a specification that relies on the OAuth2 framework to provide a well-defined authentication layer.

Simpler examples of JWTs in wild are encrypted or signed tokens that can be used to store claims on browsers and mobile clients. These claims can easily be verified by receivers through shared secrets or public keys.

The JWT specification allows for custom private claims, making JWTs a good tool for exchanging any sort of validated or encrypted data that can easily be encoded as JSON.

JWTs, like any other tool, have their own pitfalls and common attacks. The JSON Web Token Current Best Practices document attemps to enumerate them and provide clear details on how to avoid them. We will now go over the attacks and pitfalls, and later take a look at mitigations and best practices.

## Pitfalls and Common Attacks
Before taking a look at the first attack, it is important to note that many of these attacks are related to the implementation, rather than the design, of JSON Web Tokens. This does not make them less critical. It is arguable whether some of these attacks could be mitigated, or removed, by changing the underlying design. For the moment, the JWT specification and format is set in stone,so most changes happen in the implementation space (changes to libraries, APIs, programming practices and conventions).

It is also important to have a basic idea of the most common representation for JWTs: the JWS Compact Serialization format. Unserialized JWTs have two main JSON objects in them: the `header` and the `payload`.

The `header` object contains information about the JWT itself: the type of token, the signature or encryption algorithm used, the key id, etc.

The `payload` object contains all the relevant information carried by the token. There are several standard claims, like sub (subject) or iat (issued at), but any custom claims can be included as part of the payload.

These objects are encoded using the JWS Compact Serialization format to produce something like this:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.XbPfbIHMI6arZ3Y922BhjWgQzWXcXNrz0ogtVhfEd2o
```

This is a signed JWT. Signed JWTs in compact format are simply the header and payload object encoded using Base64-URL encoding and separated by a dot (`.`). The last part of the compact representation is the signature. In other words, the format is:

```
[Base64-URL encoded header].[Base64-URL encoded payload].[Signature]
```

This only applies to signed tokens. Encrypted tokens have a different serialized compact format that also relies on Base64-URL encoding and dot-separated fields.

### `"alg:none"` Attack
As we mentioned before, JWTs carry two JSON objects with important information, the `header` and the `payload`. The header includes information about the algorithm used by the JWT to sign or encrypt the data contained in it. Signed JWTs sign both the header and the payload, while encrypted JWTs only encrypt the payload (the header must always be readable).

In the case of signed tokens, although the signature does protect the header and payload against tampering, it is possible to *rewrite* the JWT without using the signature and changing the data contained in it. How does this work?

Take for instance a JWT with a certain header and payload. Something like this:
```
header: {
  alg: "HS256",
  typ: "JWT"
},
payload: {
  sub: "joe"
  role: "user"
}
```

Now, let's say you encode this token into its compact serialized form with a signature and a signing key of value "secret". We can use JWT.io for that.

Now, since this is a signed token, we are free to read it. This also means we could construct a similar token with slightly changed data in it, although in that case, we would not be able to sign it unless we knew the signing key. Let's say attackers do not know the signing key, what would they do? In this type of attack, the malicious users could attempt to use a token with no signature! How does that work?

First, the attackers modify the token. For example:
```
header: {
  alg: "none",
  typ: "JWT"
},
payload: {
  sub: "joe"
  role: "admin"
}
```

Encoded:
```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJqb2UiLCJyb2xlIjoiYWRtaW4ifQ.
```

Note that this token does not include a signature (`"alg": "none"`) and that the `role` claim of the payload has been changed. If attackers manage to use this token successfully, they may achieve an escalation of privilege attack! Why would an attack like this work? Let's take a look at how some hypothetical JWT library could work. Let's say we have a decoding function that looks like this:

```javascript
function jwtDecode(token, secret) {
    // (...)
}
```

This function takes an encoded token and a secret and attempts to verify the token and then return the decoded data in it. If verification fails, it throws an exception. To pick the right algorithm for verification, the function relies on the `alg` claim from the header. This is where the attack succeeds. In the past, many libraries relied on this claim to pick the verification algorithm, and, as you may have guessed, in out malicious token the `alg` claim is `none`. That means that there's no verification algorithm, and the verification step always succeeds.

As you can see, this is a classic example of an attack that relies on a certain ambiguity of the API of a specific library, rather than a vulnerability in the specification itself. Even so, this is a real attack that was possible in several different implementations in the past. For this reason, many libraries today report `"alg": "none"` tokens as invalid, even though there's no signature in place. There are other possible mitigations for this type of attack, the most important one being to always check the algorithm specified in the header before attempting to verify a token. Another one is to use libraries that require the verification algorithm as an input to the verification function, rather than rely on the `alg` claim.

### RS256 Public-Key as HS256 Secret Attack
This attack is similar to the `"alg": "none"` attack and also relies on ambiguity in the API of certain JWT libraries. Our sample token will be similar to the one for that attack. In this case, however, rather than removing the signature, we will construct a valid signature that the verification library will also consider valid by relying on a loophole in many APIs. First, consider the typical function signature of some JWT libraries for the verification function:

```javascript
function jwtDecode(token, secretOrPublicKey) {
  // (...)
}
```

As you can see here, this function is essentially identical to the one from the `"alg": "none"` attack. If verification is successful, the token is decoded and returned, otherwise, an exception is thrown. In this case, however, the function also accepts a public key as the second parameter. In a way, this makes sense: both the public key and the shared secret are usually strings or byte arrays, so from the point of view of the necessary types for that function argument, a single argument can represent both a public key (for RS, ES, or PS algorithm) and a shared secret (for HS algorithm). This type of function signature is common for many JWT libraries. 

Now, suppose the attacker gets an encoded token signed with an RSA key pair. It looks like this:

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqb2UiLCJyb2xlIjoidXNlciJ9.QDjcv11Kcb69THVLKMErYqzy9htWlCDtBdonVR5SX4geZa_R8StjwUuuskveUsdJVgjgXwMso7pguAJZzoE9LEr9XCxau7SF1ddws4ONiqxSVXZbO0pSgbKm3FpkVz4Jyy4oNTs-bIYyE0xf8snFlT1MbBWcG5psnuG04IEle4s
```

Decoded:

```
header: {
  "alg": "RS256",
  "typ": "JWT"
},
payload: {
  "sub": "joe",
  "role": "user"
}
```

This token is signed with an RSA key-pair.RSA signatures are produced with the private key, while verification is done with the public key. Whoever verifies the token in the future could make a call to out hypothetical `jwtDecode` function from before like so:

```
const publicKey = '...';
const decoded = jwtDecode(token, publicKey);
```

But here's the problem: the public key is, like the name implies, usually public. Attackers may get their hands on it, and that should be OK. But what if the attackers were to create a new token using the following scheme. First, the attackers modify the header and choose HS256 as the signing algorithm:
```
header: {
  "alg": "HS256",
  "typ": "JWT"
}
```

Then they escalate permisssion by changing the `role` claim in the payload:
```
payload: {
  "sub": "joe",
  "role": "admin"
}
```

Now, here's the attack: the attacker proceeds to create a newly encoded JWT by using the public key, which is a simple string, as the HS256 shared secret! In other words, since the shared secret for HS256 can be any string, even a string like the public key for the RS256 algorithm can be used for that.

Now if we go back to our hypothetical use of the `jwtDecode` function from before:
```javascript
const publicKey = '....';
const decoded = jwtDecode(token, publicKey);
```

We can now clearly see the problem, the token will be considered valid! The public key will get passed to the `jwtDecode` function as the second argument, but rather than being used as a public key for the RS256 algorithm, it will be used as a shared secret for the HS256 algorithm. This is caused by the `jwtDecode` function relying on the `alg` claim from the header to pick the verification algorithm for the JWT. And the attacker changed that.
```
header: {
  "alg": "HS256", // <-- changed by the attacker from RS256
  "typ": "JWT"
}
```

Just like in the `"alg": "none"` case, relying on the alg claim combined with a bad or confusing API can result in a successful attack by a malicious user.

Mitigations against this attack include passing an explicit algorithm to the jwtDecode function, checking the alg claim, or using APIs that separate public-key algorithms from shared secret algorithm.

### Weak HMAC Keys
HMAC algorithms rely on a shared secret to produce and verify signatures. Some people assume that shared secrets are similar to passwords, and in a sense, they are: they should be kept secret. However, that is where the similarities end. For passwords, although the length is an important property, the minimum required length is relatively small compared to other types of secrets. This is a consequence of hashing algorithms that are used to store passwords (along with a salt) that prevent brute force attacks in reasonable timeframes.

On other hand, HMAC shared secrets, as used by JWTs, are optimized for speed. This allows many sign/verify operations to be performed efficiently but make brute force attacks easier. So, the length of the shared secret for HS256/384/512 is of the utmost importance. In fact, JSON Web Algorithms defines the minimum key length to be equal to the size in bits of the hash function used along with the HMAC algorithm:

>"A key of the same size as the hash output (for instance, 256 bits for "HS256") or larger MUST be used with this algorithm."  - JSON Web Algorithms (RFC 7518), 3.2 HMAC with SHA-2 Functions

In other words, many passwords that could be used in other contexts are simply not good enough for use with HMAC-signed JWTs. 256-bits equals 32 ASCII characters, so if you are using something human readable, consider that number to be the minimum number of characters to include in the secret. Another good option is to switch to RS256 or other public-key algorithms, which are much more robust and flexible. This is not simply a hypothetical attack, we have shown that brute force attacks for HS256 are simple enough to perform if the shared secret is too short.

### Wrong Stacked Encryption + Signature Verification Assumptions
Signatures provide protection against tampering. That is, although they don't protect data from being readable, they make it immutable: any changes to the data result in an invalid signature. Encryption, on the other hand, makes data unreadable unless you know the shared key or private key.

For many applications, signatures are all that is necessary. However, for sensitive data, encryption may be required. JWTs support both: signatures and encryption.

It is very common to wrongly assume that encryption also provides protection against tampering in all cases. The rationale for this assumption is usually something like this: "if the data can't be read, how would an attacker be able to modify it for  their benefit?". Unfortunately, this underestimates attackers and their knowledge of the algorithm involved in the process.

Some encryption/decryption algorithms produce output regardless of the validity of the data passed to them. In other words, even if the encrypted data was modified, something will come out of the decryption process. Bindly modifying data usually results in garbage as output, but to a malicious attacker, this may be enough to get access to a system. For example, consider a JWT payload that looks like this:

```json
{
  "sub": "joe",
  "admin": false
}
```

As we can see here, the `admin` claim is simply a boolean. If attackers can manage to produce a change in the decrypted data that results in that boolean value being flipped, they may successfully execute an escalation of privileges attack. In particular, attackers can try as many changes to encrypted data as they like, without having the system discard the token as invalid before processing it. Other attacks may involve feeding invalid data to subsystems that expect data to be already sanitized at that point, triggering bugs, failures, or serving as the entry point for other types of attacks.

For this reason, JSON Web Algorithms only defines encryption algorithm that also include data integrity verification. In other words, as long as the encryption algorithm is one of the algorithms sanctioned by JWA, it may not be necessary for your application to stack an encrypted JWT on top of a signed JWT. However, if you encrypt a JWT using a non-standard algorithm, you must either make sure that data integrity is provided by that algorithm, or you will need to nest JWTs, using a signed JWT as the innermost JWT to ensure data integrity.

Nested JWTs are explicitly defined and supported by the specification. Although unusual, they may also appear in other scenarios, like sending a token issued by some party through a third party system that also uses JWTs.

A common mistake in these scenarios is related to the validation of the nested JWT. To make sure that data integrity is preserved, and that data is properly decoded, all layers of JWTs must pass all validations related to the algorithms defined in their headers. In other words, even if the outermost JWT can be decrypted and validated, it is also necessary to validate (or decrypt) all the innermost JWTs. Failing to do so, especially in the case of an outermost encrypted JWT carrying an innermost signed JWT, can result in the use of unverified data, with all the associated security issues related to that.

![Alt text](images/image-jwt.png)

### Invalid Elliptic-Curve Attacks
Elliptic-curve cryptography is one of the public-key algorithm families supported by JSON Web Algorithms. Elliptic-curve cryptography relies on the intractability of the elliptic-curve discrete logarithm problem, a mathematical problem that cannot be solve in reasonable times for big enough numbers. This problem prevents the recovery of the private key from a public key, an encrypted message, and its plaintext. When compared to RSA, another public-key algorithm which is also supported by JSON Web Algorithms, elliptic-curves provide a similar level of strength while requiring smaller keys.

Elliptic-curves, as required for cryptographic operations, are defined over finite fields. In other words, they operate on sets of discrete numbers (rather than all real numbers). This means that all numbers involved in cryptographic elliptic-curve operations are integers.

All methematical operations of elliptic-curves result in valid points over the curve. In other words, the math for elliptic-curves is defined in such a way that invalid points are simply not possible. If an invalid point is produced, then there is an error in the inputs to the operations. The mains arithmetic operations on elliptic curves are: 
- Point addition: adding two points on the same curve resulting in a third point on the same curve.
- Point doubling: adding a point to itself, resulting in a new point on the same curve.
- Scalar multiplication: multiplying a single point on the curve by a scalar number, defined as repeatedly adding that number to itself k times (where k is the scalar value).

All cryptographic operations on elliptic-curves rely on these arithmetic operations. Some implementations, however, fail to validate the inputs to them. In elliptic-curve cryptography, the public key is a point on the elliptic curve, while the private key is simply a number that sits within a special, but very big, range. If inputs to these operations are not validated, the arithmetic operation may produce seemingly valid results even when they are not. These results, when used in the context of cryptographic operations such as decryption, can be used to recover the private key. This attack has been demonstrated in the past. This class of attacks is known as invalid curve attacks. Good-quality implementations always check that all inputs passed to any public function are valid. This includes verifying that public-keys are a valid elliptic-curve point for the chosen curve and that private keys inside the valid range of values.

### Substitution Attacks
Substitution attacks are a class of attacks where an attacker manages to intercept at least two different tokens. The attacker then manages to use one or both of these tokens for purposes other than the one they were intended for.

There are 2 types of substitution attacks: same recipient (called cross JWT in the draft), and different recipient.

**<u>Different Recipient</u>**
Different recipient attacks work by sending a token intended for one recipient to a different recipient. Let's say there is an authorization server that issues tokens for a third party service. The authorization token is signed JWT with the following payload:

```
{
  "sub": "joe",
  "role": "admin"
}
```

This token can be used against an API to perform authenticated operations. Furthermore, at least when it comes to this service, the user `joe` has administrator level privileges. However, there is a problem with this token: there is no intended recipient or even an issuer in it. What would happen if a different API, different from the intended recipient this token was issued for, used the signature as the only check for validity? Let's say there's also a user `joe` in the database for that service or API. The attacker could send this same token to that other service and instantly gain administrator privileges!

![Alt text](images/image-diff-recipient.png)

To prevent these attacks, token validation must rely on either unique, per-service keys or secrets, or specific claims. For instance, this token could include an aud claim specifying the intended audience. This way, even if the signature is valid, the token cannot be used on other services that share the same secret or signing key.

**<u>Same Recipient/Cross JWT</u>**
This attack is similar to the previous one, but rather than relying on a token issued for a different recipient, in this case, the recipient is the same. What changes, in this case, is that the attacker sends the token to a different service rather the one intended for (inside the same company or service provider). Let imagine a token with the following payload:
```
{
    "sub": "joe",
    "perms": "write",
    "aud": "cool-company/user-database",
    "iss": "cool-company"
}
```

This token looks much more secure. We have an issuer (iss) claim, an audience (aud) claim, and a permissions (perm) claim. The API for which this token was issued checks all of these claims even if the signature of the token is valid. This way, even if attackers manage to get their hands on a token signed with the same private key or secret, they cannot use it to operate on this service if it's not intended for it.

However, `cool-company` has other public services. One of these services, the `cool-company/item-database` service, has recently been upgraded to check claims along with the token signature. However, during the upgrades, the team in charge of selecting the claims that would be validated made a mistake: they did not validate the aud claim correctly. Rather than checking for an exact match, they decided to check for the presence of the `cool-company` string. It turns out that the other service, the hypothetical `cool-company/user-database` service, emits tokens that also pass this check. In other words, attackers could use the token intended for the `user-database` service in place for the token for the `item-database` service. This would grant the attackers write permissions to the item database when they should only have write permissions for the user database!

![Alt text](images/image-same-recipient.png)

## Mitigations and Best Practices

#### Always Perform Algorithm Verification
The `"alg": "none"` attack and the "RS256 public-key as HS256 shared secret" attack can be prevented by this mitigation. Every time a JWT is to be validated, the algorithm must be explicitly selected to prevent giving attackers control. Libraries used to rely on the header alg claim to select the algorithm for validation. From the moment attacks like these were seen in the wild, libraries have switched to at least provide the option of explicitly specifying the selected algorithm for validation, disregarding what is specified in the header. Still, some libraries provide the option of using whatever is specified in the header, so developers must take care to always use explicit algorithm selection.

#### Use Appropriate Algorithm
Although the JSON Web Algorithm spec declares a series of recommended and required algorithm, picking the right one for a specific scenario is still up to the users. For example, a JWT signed with an HMAC signature may be enough for storing a small token from your single-server, single-page web application in a user's browser. In constrast, a shared secret algorithm would be sorely inconvenient in a federated identity scenario.

Another way of thinking about this is to consider all JWTs invalid unless that validation algorithm is acceptable to the application. In other words, even if the validating party has the keys and the means necessary for validating a token, it should still be considered invalid if the validation algorithm is not the right one for the application. This is also another way of saying what we mentioned in our previous recommendation: always perform algorithm verification.

#### Always Perform All Validations
In the case of nested tokens, it is necessary to always perform all validation steps as declared in the headers of each token. In other words, it is not sufficient to decrypt or validate the outermost token and then skip validation for the inner ones. Even in the case of only having signed JWTs, it is necessary to validate all signatures. This is source of common mistakes in applications that use JWTs to carry other. JWTs issued by external parties.

#### Always Validate Cryptographic Inputs 
