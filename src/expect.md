# Expect-Based Attacks

RFC 7231 Expect header behavior exploited when proxies implement it inconsistently

## Vanilla Expect: 100-continue >>> RQP
- Client sends: `Expect: 100-continue`
- Server responds: `100 Continue` or error
- Client then sends body
- Some proxies don't implement properly

## Basic Expect Test Request
```
POST / HTTP/1.1
Host: example.com
Content-Length: 60
Expect: 100-continue
(wait for 100 Continue)
GET /404 HTTP/1.1
Host: example.com
```

## Basic Expect Results
- 100 Continue then 200 OK >>> Both servers implement Expect correctly
  - No desync vulnerability
  - Try obfuscation
- Timeout then 417 Expectation Failed >>> Back-end doesn't support Expect
  - No desync
  - Try different technique
- 200 OK immediately >>> Front-end processed body without waiting
  - Potential desync
  - Next step >>> Obfuscated Expect
- Two responses (200, 404) >>> Desync successful
  - Front-end waited, back-end didn't
  - Next step >>> RQP || Obfuscated Expect

## Obfuscated Expect Headers >>> RQP
- Hide Expect from one parser
- V-H variants (hide from back-end):
  - ` Expect: 100-continue`
  - `Expect : 100-continue`
  - `Expect: 100-continue `
- H-V variants (hide from front-end):
  - Same obfuscations but different parser tolerance

## Obfuscated Test Request
```
POST / HTTP/1.1
Host: example.com
Content-Length: 60
 Expect: 100-continue

GET /404 HTTP/1.1
Host: example.com
```

## Obfuscated Results
- Immediate response without 100 Continue >>> Front-end didn't see Expect
  - Sent body immediately
  - Back-end may have waited
- Back-end processed 100-continue >>> Back-end saw Expect
  - Waited for body
  - Desync condition met
- Two responses >>> Desync successful
  - Next step >>> RQP || Response header bypass

## Response Header Bypass >>> RQP
- Some parsers check response headers for Expect
- If `Expect: 100-continue` in response → treat as continuation
- Inject via response smuggling

## Response Header Test
- Chain with existing desync
- Smuggle response containing: `Expect: 100-continue`
- Causes parser confusion on next request

## Response Header Results
- Parser treats response as continuation >>> Bypass successful
  - Next request gets merged
  - Next step >>> RQP
- Next request gets merged >>> Desync chain established
  - Can control subsequent request parsing
  - Full exploitation possible
- Can control subsequent request parsing >>> Full exploitation
  - Session hijacking
  - Cache poisoning

## Expect with Content-Length: 0 >>> RQP
- Edge case: Expect with no body

## CL:0 Expect Test Request
```
POST / HTTP/1.1
Expect: 100-continue
Content-Length: 0
```
- Some servers send body buffer anyway

## CL:0 Expect Results
- 100 Continue sent >>> Server expects body despite CL:0
  - Parser confusion
  - Potential desync
- Immediate 200 >>> Server processed correctly
  - No vulnerability
  - Try different technique
- Timeout >>> Parser confusion
  - Potential desync
  - Try sending body anyway
