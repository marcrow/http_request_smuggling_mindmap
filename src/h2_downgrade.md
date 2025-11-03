# HTTP/2 Downgrade Attacks

## Definition
- HTTP/2 front-end downgrades to HTTP/1.1 back-end creating parsing discrepancies

## H2.CL Desync >>> RQP
- Front-end doesn't forward Content-Length (uses H2 framing)
- Back-end uses Content-Length from H2 pseudo-header
- Result: CL.0-like behavior

## H2.CL Test
- Send via HTTP/2: `:method: POST`, `:path: /`, `:authority: example.com`, `content-length: 0`
- Body: `GET /404 HTTP/1.1\r\nHost: example.com\r\n\r\n`

## H2.CL Results
- Single 200 OK >>> No desync
  - Back-end ignored body
  - Try different technique
- Two responses (200, 404) >>> Desync successful
  - H2 front-end treated body as complete
  - H1 back-end expected Content-Length
  - Back-end processed smuggled GET
  - Next step >>> RQP || Cache poison

## H2.TE Desync >>> RQP
- Front-end strips Transfer-Encoding (not valid in H2)
- Back-end receives smuggled TE header in body
- Back-end processes chunked encoding

## H2.TE Test
- Send via HTTP/2: `:method: POST`, `:path: /`
- Body: `POST / HTTP/1.1\r\nTransfer-Encoding: chunked\r\n\r\n5c\r\nGPOST /404 HTTP/1.1\r\n...\r\n0\r\n\r\n`

## H2.TE Results
- 404 for GPOST >>> Desync successful
  - Chunked encoding processed by back-end
  - Front-end forwarded entire body
  - Back-end parsed smuggled TE header
  - Next step >>> RQP
- Single 200 >>> No desync
  - Back-end didn't process TE
  - Try H2.CL or H2.0

## H2.0 Desync >>> RQP
- Combines H2.CL and H2.TE concepts
- HTTP/2 request with no Content-Length or TE
- Body contains complete HTTP/1.1 request
- Back-end interprets based on H1 headers in body

## H2.0 Test
- Send via HTTP/2 (no CL or TE): `:method: POST`, `:path: /`
- Body: `GET /404 HTTP/1.1\r\nHost: example.com\r\n\r\n`

## H2.0 Results
- Two responses >>> Back-end parsed body as separate request
  - Desync successful
  - Next step >>> RQP
- Single response >>> Back-end ignored body
  - Connection handling varies
  - Try H2.CL or H2.TE

## HTTP/2 Header Injection >>> RQP
- Pseudo-headers in H2 become real headers in H1
- Smuggle headers via H2 pseudo-headers
- `:path: /test\r\nX-Smuggled: header`
- Becomes: `GET /test\r\nX-Smuggled: header HTTP/1.1`

## Header Injection Results
- X-Smuggled appears in back-end >>> Header injection successful
  - Can inject Host, Content-Length, etc.
  - Full header control
  - Next step >>> Cache poison || XSS gadget
- Can inject Host, Content-Length, etc. >>> Full header control
  - Back-end vulnerable to CRLF
  - Additional attack surface
  - Next step >>> RQP

## H2 Connection Reuse >>> RQP
- Front-end reuses H2 connections
- Back-end uses H1 connection pool
- Desync poisons back-end connection
- Affects subsequent requests on same connection

## Connection Reuse Results
- Victim requests get poisoned >>> Connection pool poisoning
  - Persistent desync state
  - Affects multiple users
- Persistent desync state >>> Affects multiple users
  - More reliable than single-shot attacks
  - Higher impact
  - Next step >>> RQP
