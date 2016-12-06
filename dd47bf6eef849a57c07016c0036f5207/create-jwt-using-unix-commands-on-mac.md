### Pseudocode: 
```
Y = Base64URLEncode(Header) + ‘.’ + Base64URLEncode(Payload)
JWT = Y + ‘.’ + Base64URLEncode(HMACSHA256(Y))
```
> The steps called out here should work on a Mac as well. The only thing that might be different is the `sed` command used below. Instead of using `-E`, you will have to use `-r` to run `sed` with extended regular expression support

### Use data from this tutorial:
[scotch.io](https://scotch.io/tutorials/the-anatomy-of-a-json-web-token)

### Header:
```javascript
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Base64 Encode of Header:
`echo -n '{"alg":"HS256","typ":"JWT"}' | openssl base64 -e -A`

OR

`echo -n '{"alg":"HS256","typ":"JWT"}' | base64`

`Output: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

> However, we need Base64 URL encoding of the Header.

### Base64 URL Encoding of Header:
`echo -n '{"alg":"HS256","typ":"JWT”}' | openssl base64 -e -A | sed s/\+/-/ | sed -E s/=+$//`

OR

`echo -n '{"alg":"HS256","typ":"JWT”}' | base64 | sed s/\+/-/ | sed -E s/=+$//`

`Output: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

> Repeat the same series of steps for Payload.

### Payload:
```javascript
{
  "iss": "cisco.com",
  "exp": 1470839345,
  "name": "Anand Sharma",
  "cda-admin": true
}
```

### Base64 Encoding of Payload:
`echo -n '{"iss":"cisco.com","exp":1470839345,"name":"Anand Sharma","cda-admin":true}' | openssl base64 -e -A`

OR

`echo -n '{"iss":"cisco.com","exp":1470839345,"name":"Anand Sharma","cda-admin":true}' | base64`

`Output: eyJpc3MiOiJjaXNjby5jb20iLCJleHAiOjE0NzA4MzkzNDUsIm5hbWUiOiJBbmFuZCBTaGFybWEiLCJjZGEtYWRtaW4iOnRydWV9`

### Base64 URL Encoding of Payload:
`echo -n '{"iss":"cisco.com","exp":1470839345,"name":"Anand Sharma","cda-admin":true}' | openssl base64 -e -A | sed s/\+/-/ | sed -E s/=+$//`

OR

`echo -n '{"iss":"cisco.com","exp":1470839345,"name":"Anand Sharma","cda-admin":true}' | base64 | sed s/\+/-/ | sed -E s/=+$//`

`Output: eyJpc3MiOiJjaXNjby5jb20iLCJleHAiOjE0NzA4MzkzNDUsIm5hbWUiOiJBbmFuZCBTaGFybWEiLCJjZGEtYWRtaW4iOnRydWV9`

###Y:
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJjaXNjby5jb20iLCJleHAiOjE0NzA4MzkzNDUsIm5hbWUiOiJBbmFuZCBTaGFybWEiLCJjZGEtYWRtaW4iOnRydWV9`

###HMAC SHA256 Digest (Default output in Hex):
`echo -n 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJjaXNjby5jb20iLCJleHAiOjE0NzA4MzkzNDUsIm5hbWUiOiJBbmFuZCBTaGFybWEiLCJjZGEtYWRtaW4iOnRydWV9' | openssl dgst -sha256 -hmac secret`

`Output: 21557ae7825781d1176595f7ce506b96e3e02fc0711564d02abb4722c3be5eb5`

> This is where the problem starts. What we forget is that the openssl dgst command is dumping the hexadecimal encoded version of the digest, which is really a binary data. So if you pass this value (the hexadecimal output shown above) to the base64 encoding command, you will get Base64 encoding of basically "plain text data”. Why? Because it is going to treat the hexadecimal output as plain text (or string). What we need is to feed the binary data into base64 encoding command. Btw, this [StackOverflow](http://stackoverflow.com/questions/32188149/difference-between-cryptojs-enc-base64-stringify-and-normal-base64-encryption) question and answer was the thing that _really_ helped me out. Plus, the [Swiss Converter Tool](http://www.percederberg.net/tools/text_converter.html) allowed me to actually confirm that my understanding was correct

### Base64 URL Encode the HMAC SHA256 Digest (Output in Binary):
`echo -n 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJjaXNjby5jb20iLCJleHAiOjE0NzA4MzkzNDUsIm5hbWUiOiJBbmFuZCBTaGFybWEiLCJjZGEtYWRtaW4iOnRydWV9’ | openssl dgst -sha256 -hmac secret -binary | openssl base64 -e -A | sed s/\+/-/ | sed -E s/=+$//`

`Output: IVV654JXgdEXZZX3zlBrluPgL8BxFWTQKrtHIsO-XrU`

### JWT:
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJjaXNjby5jb20iLCJleHAiOjE0NzA4MzkzNDUsIm5hbWUiOiJBbmFuZCBTaGFybWEiLCJjZGEtYWRtaW4iOnRydWV9.IVV654JXgdEXZZX3zlBrluPgL8BxFWTQKrtHIsO-XrU`

> Go to [jwt.io](http://jwt.io) and verify the JWT token, including the signature. 

![Screenshot from jwt.io](https://raw.githubusercontent.com/indrayam/media-files/master/jwt-token-verification.jpg)

> Yippie! :sparkles: :boom: :smile: