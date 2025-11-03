# CL.0 Desync Attacks

Back-end doesn't see Content-Length while Front-end processes it normally (V-H discrepancy)

## CL.0 Basic >>> RQP
- Hide CL from back-end: ` Content-Length: 30`

## Basic Test Request
```
POST / HTTP/1.1
Host: example.com
 Content-Length: 30

GET /404 HTTP/1.1
Host: example.com
```

## Basic Test Results
- Single 200 OK >>> No desync
  - Back-end saw Content-Length
  - Try different obfuscation
- Two responses (200, 404) >>> Desync successful
  - Back-end processed GET separately
  - Next step >>> RQP || Early-response gadget
- Timeout >>> Back-end waiting
  - Waiting for more headers
  - Try early-response gadget

## CL.0 with Early-Response Gadget >>> RQP || Double desync
- Some back-ends respond early without reading body
- Use early-response endpoints to break deadlock
- Common gadgets:
  - `/con` - Console endpoints
  - `/nul` - Null/status endpoints
  - `/admin` - 401/403 immediate responses
  - `/static/*` - Early 404 responses

## Early-Response Test Request
```
POST /con HTTP/1.1
Host: example.com
 Content-Length: 60

GET /404 HTTP/1.1
Host: example.com

GET /test HTTP/1.1
Host: example.com
```

## Early-Response Results
- Single response >>> No early gadget found
  - Try different endpoints
  - Check /admin, /api, /health
- Multiple responses >>> Early response triggered
  - Body left unread
  - Next step >>> Double desync || Cache poison
- Second request gets 404 >>> Desync successful
  - Prefix poisoned next request
  - Next step >>> RQP

## Double Desync >>> RQP
- Chain two CL.0 desyncs
- First: Poison back-end socket
- Second: Poison front-end socket
- Bypass keep-alive limits

## Double Desync Test
- Request 1: CL.0 with early gadget
- Request 2: Normal request (gets prefix)
- Request 3: Another CL.0 (poisons front-end)
- Result: Full control over response queue

## Double Desync Results
- Responses out of order >>> Double desync working
  - Full control over response queue
  - Next step >>> RQP
- Victim request gets attacker's prefix >>> RQP successful
  - Can capture sensitive data
  - Can inject headers
- Can inject headers >>> Full exploitation possible
  - Session hijacking
  - Cache poisoning
