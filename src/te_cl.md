# TE.CL Desync Attacks

Front-end uses Transfer-Encoding while Back-end uses Content-Length

## Classic TE.CL >>> RQP
- Hide Content-Length from back-end
  - V-H discrepancy exploitation
    - Leading space: ` Content-Length: 4`
    - Upper-Lower: ` Content-length: 4`
    - Tab: ` Content-Length:\n4`

## How to exploit when vulnerable

- CL should cover only the first chunk definition line + return line
- HTTP/1.1
- Do not forget CL for embded POST request (set it with high value)
- Finnish the request with 0\r\n\r\n


## Test Request

- POST request example
  - 
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
  - Switch to HTTP/1.1
  - Lock Content-Length
  - Add your body 
  - Select only the body > ext > convert to chunked
  - Change CL (nb char for chunk lentgh + 2) 
  - If the embded request is a POST, CL = 2nd body + 6
    - Note: add more CL is more secured 
  - Finnish the request per 0\r\n\r\n
    -  'Smuggle attack TE.CL' via the Smuggler extension. 
- If you want to test with embdeed GET request
  - Add a body
  - Add a large CL for the Get request
   
## Expected Results
- Response code vary >>> desync
- 200 OK with normal page >>> No desync
  - Front-end blocked request
  - Back-end saw Content-Length header
- 404 Not Found for /gpost >>> Desync successful
  - Back-end processed GPOST as separate request
  - Next step >>> RQP || Cache poison || XSS gadget
- Timeout >>> Back-end waiting
  - Waiting for more data based on Content-Length
  - Adjust Content-Length value
  - Are you sure to correctly form the request (\r\n\r\n at the end)

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
