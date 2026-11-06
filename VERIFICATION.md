# Verification log

## 2026-11-06: CORS
- GET /vehicles/GetMakesForVehicleType/car?format=json with Origin: https://southpaw199.github.io
- Response header: access-control-allow-origin: *
- Result: browser calls allowed; Swagger UI Try it out should work on GitHub Pages. Confirm in browser during S5.

## 2026-11-06: DecodeVinValues, partial VIN
- GET /vehicles/DecodeVinValues/5UXWX7C5*BA?format=json&modelyear=2011
- Fixture: fixtures/decodevinvalues-partial-vin.json
- Make BMW, Model X3, ModelYear "2011" (string), ErrorCode "6", ErrorText "6 - Incomplete VIN"
- Note: nonzero ErrorCode returned alongside a usable decode; numeric values come back as strings.
