# Detection & Initial Tests
This is a description test 2

## Baseline Request
- Default HTTP request
  - `GET /icon.svg HTTP/1.1`
  - `Host: example.com`

## Parser Discrepancy Detection >>> V-H discrepancy || H-V discrepancy
- Use HTTP Request Smuggler v3.0
  - Test with obfuscated headers: Content-Length, Transfer-Encoding, Host, Expect, Max-Forwards, Range

## V-H Discrepancy Interpretation
- Visible-Hidden (V-H): Front-end sees header, back-end doesn't
  - Response analysis
    - `Host: example.com` → 200 OK (normal)
    - `Xost: example.com` → 503 Service Unavailable (arbitrary masked)
    - ` Host: example.com` → 400 Bad Request (unique response!)
    - Interpretation: Leading space visible to front-end, hidden from back-end
  - Exploitation path >>> CL.0 desync || TE.CL desync
    - CL.0: Hide Content-Length from back-end
    - TE.CL: Hide Transfer-Encoding from back-end

## H-V Discrepancy Interpretation
- Hidden-Visible (H-V): Front-end doesn't see header, back-end does
  - Response analysis
    - `Host: foo/bar` → 400, Server: awselb/2.0 (front-end error)
    - ` Host: foo/bar` → 400, Server: Microsoft-HTTPAPI/2.0 (back-end error!)
    - Interpretation: Leading space hides from front-end, visible to back-end
  - Exploitation path >>> CL.TE desync || 0.CL desync
    - CL.TE: Hide Transfer-Encoding from front-end
    - 0.CL: Hide Content-Length from front-end

## Detection Strategies
- Single strategy (normal Host)
  - `Host: example.com` → 200 OK
- Duplicate strategy (invalid value)
  - `Host: example.com`
  - `Host: x/x` → 400 Bad Request
  - ` Host: x/x` → 200 OK (discrepancy detected!)
- GET vs POST methods
  - Some front-ends reject GET with body
  - Switch to OPTIONS if needed
