# Verification log

## 2026-11-06: CORS verified
- Ran a verification from a Swagger UI scratch page running on localhost (http://localhost:8080) and used DevTools > Console:
    ```
    fetch("https://vpic.nhtsa.dot.gov/api/vehicles/GetMakesForVehicleType/car?format=json").then(r => r.json()).then(d => console.log(d.Count, d.Message))
- Output with the count and message:
  `195 'Response returned successfully'`
- Result: Browser cross-origin GET succeeded without a CORS error.
- Action: Need to re-run when published on GitHub Pages in S6.

## 2026-11-06: CORS
- GET /vehicles/GetMakesForVehicleType/car?format=json with Origin: https://southpaw199.github.io
- Response header: access-control-allow-origin: *
- Result: browser calls allowed; Swagger UI Try it out should work on GitHub Pages. Confirm in browser during S5.

## 2026-11-06: DecodeVinValues, partial VIN
- GET /vehicles/DecodeVinValues/5UXWX7C5*BA?format=json&modelyear=2011
- Fixture: fixtures/decodevinvalues-partial-vin.json
- Make BMW, Model X3, ModelYear "2011" (string), ErrorCode "6", ErrorText "6 - Incomplete VIN"
- Note: nonzero ErrorCode returned alongside a usable decode; numeric values come back as strings.
