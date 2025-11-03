# 0.CL Desync Attacks

Front-end doesn't see Content-Length while Back-end processes it normally (H-V discrepancy)

## 0.CL Basic >>> RQP
- Hide CL from front-end: ` Content-Length: 30`
- Front-end forwards immediately (no body expected)
- Back-end waits for body

## Deadlock Problem
- Front-end sends headers immediately
- Back-end waits for Content-Length bytes
- Connection enters deadlock state
- Must break deadlock to exploit

## Breaking Deadlock with Early-Response Gadgets >>> RQP
- Find endpoints that respond before reading body
- Common patterns:
  - `/con` `/nul` - Console/status endpoints
  - Redirect endpoints (301/302)
  - Authentication failures (401/403)
  - Rate limit responses (429)
  - Method not allowed (405)

## Early-Response Test Request
```
POST /con HTTP/1.1
Host: example.com
 Content-Length: 60

GPOST /test HTTP/1.1
Host: example.com
Content-Length: 10

x=1
```

## Early-Response Results
- Timeout >>> No early-response gadget found
  - Try different endpoints
  - Check /admin, /api, /health, /status
- Immediate response + 404 for GPOST >>> Desync successful
  - Back-end processed GPOST separately
  - Next step >>> RQP || Double desync || Cache poison
- Connection closed >>> Server detected anomaly
  - Try different obfuscation
  - Look for alternative gadgets

## Double Desync Technique >>> RQP
- Chain two 0.CL desyncs
- First request: Poison back-end connection
- Second request: Poison front-end connection
- Achieve full control over response queue

## Double Desync Test Sequence
- Request 1: 0.CL with early gadget
  - Back-end responds early
  - GPOST prefix left in back-end buffer
- Request 2: Normal request from victim
  - Gets poisoned with GPOST prefix
  - Response goes to attacker
- Request 3: Another 0.CL attack
  - Poisons front-end connection

## Double Desync Results
- Victim gets 404 >>> Their request was prefixed with GPOST
  - Request parsing modified
  - Next step >>> RQP
- Attacker receives victim's response >>> Response queue poisoned
  - Can read sensitive data
  - Session tokens exposed
- Responses out of order >>> Full desync achieved
  - Complete control over response queue
  - Can inject arbitrary headers
- Can inject arbitrary headers >>> Complete exploitation
  - Session hijacking
  - Cache poisoning

## Alternative Deadlock Breaking
- Some servers have timeout mechanisms
- TCP keep-alive may trigger response
- Connection pooling edge cases
- Server-side request timeouts
