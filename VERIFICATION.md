# Verification log

## Notes

- APIs are published at [NHTSA Vehicle API](https://vpic.nhtsa.dot.gov/api/).
- The response envelope consists of:
  - `Count`: a number, the number of items in `Results`
  - `Message`: a status string. It is not a reliable success flag: bad input also says "Results returned successfully".
  - `SearchCriteria`: echoes the request as a string, but can be `""` (batch) or `null` (errors).
  - `Results`: an array, which can be empty.
- Field Types:
  - Empty values come back as `""`.
  - DecodeVinValues returns every value as a string (ModelYear: "2003").
  - GetMakesForVehicleType and GetModelsForMakeYear return IDs as numbers.
  - The same concept has different names in the two: `MakeId/MakeName` versus `Make_ID/Make_Name`.
- Error Behavior
  - Bad values return 200, with errors in the body. Look for `ErrorCode`/`ErrorText`, `Count 0`, or "Execution Error".
  - Wrong paths return a real 404 with a `{"message": ...}` body that has a different shape.
- JSON fixtures are pretty-printed with jq . (whitespace only).

## 2026-11-08: DecodeVinValues - Valid VIN

- GET /DecodeVinValues/1HGCM82633A004352?format=json
- Fixture: fixtures/decodevinvalues-full-vin.json
- Response: 200 application/json
- Count: `1`
- SearchCriteria: `"VIN(s): 1HGCM82633A004352"`

## 2026-11-08: DecodeVinValues: Partial VIN

- GET /DecodeVinValues/1HGCM?format=json
- Fixture: fixtures/decodevinvalues-partial-vin-short.json
- Response: `200 application/json`
- Count: `1`
- SearchCriteria: `"VIN(s): 1HGCM"`
- Returns all fields, populating only those that match the search criteria.
- AdditionalErrorText: `""`
- ErrorCode: `"6,8,11"`
- ErrorText: `"6 - Incomplete VIN; 8 - No detailed data available currently; 11 - Incorrect Model Year - Position 10 does not match valid model year codes (I, O, Q, U, Z, 0). Decoded data may not be accurate."`

## 2026-11-08: DecodeVinValues: Invalid VIN

I tried just numeric characters:

- GET /DecodeVinValues/12345?format=json
- Fixture: fixtures/decodevinvalues-invalid-vin.json
- Response: `200 application/json`
- Count: `1`
- Message: No indication of error.
- SearchCriteria: `"VIN(s): 12345"`
- AdditionalErrorText: `""`
- ErrorCode: `"6,8,11"`
- ErrorText: `"6 - Incomplete VIN; 8 - No detailed data available currently; 11 - Incorrect Model Year - Position 10 does not match valid model year codes (I, O, Q, U, Z, 0). Decoded data may not be accurate."`

I tried non-alphanumeric characters:

- GET /DecodeVinValues/$$$$$?format=json
- Fixture: fixtures/decodevinvalues-invalid-vin-special-characters.json
- Response: `200 application/json`
- Count: `1`
- Message: No indication of error.
- SearchCriteria: `"VIN(s): $$$$$"`
- AdditionalErrorText: `"Invalid character(s): 1:$, 2:$, 3:$, 4:$, 5:$."`
- ErrorCode: `"6,7,11,400"`
- ErrorText: `"6 - Incomplete VIN; 7 - Manufacturer is not registered with NHTSA for sale or importation in the U.S. for use on U.S roads; Please contact the manufacturer directly for more information; 11 - Incorrect Model Year - Position 10 does not match valid model year codes (I, O, Q, U, Z, 0). Decoded data may not be accurate.; 400 - Invalid Characters Present"`
- Note: First attempt of this operation in a bash shell using `curl` with double quotes resulted in the shell expanding `$$` to its process ID, so vPIC received a number instead of the VIN. Re-ran it successfully with single quotes.

I tried a mix of non-alphanumeric characters:

- GET /DecodeVinValues/$$^$!$$?format=json
- Fixture: fixtures/decodevinvalues-invalid-vin-mixed-special-characters.json
- Response: `200 application/json`
- Count: `1`
- Message: No indication of error
- SearchCriteria: `"VIN(s): $$^$!$$"`
- AdditionalErrorText: `"Invalid character(s): 1:$, 2:$, 3:^, 4:$, 5:!, 6:$, 7:$."`
- ErrorCode: `"6,7,11,400"`
- ErrorText: `"6 - Incomplete VIN; 7 - Manufacturer is not registered with NHTSA for sale or importation in the U.S. for use on U.S roads; Please contact the manufacturer directly for more information; 11 - Incorrect Model Year - Position 10 does not match valid model year codes (I, O, Q, U, Z, 0). Decoded data may not be accurate.; 400 - Invalid Characters Present"`
- Note: First attempt, like the operation with the `$$$$$`, sending this operation in double quotes resulted in the shell expansion for the `$$` plus a bash history expansion for the `!`. Re-ran it successfully with single quotes.

## 2026-11-08: DecodeVinValues: No VIN

- GET /DecodeVinValues/?format=json
- Fixture: fixtures/decodevinvalues-missing-vin.json
- Received this response: `404 application/json; charset=utf-8`
- Message: `"No HTTP resource was found that matches the request URI 'https://backend-vpic-api.nhtsa.dot.gov/api/vehicles/DecodeVinValues/?format=json'."`

## 2026-11-08: DecodeVinValues: No Format

- GET /DecodeVinValues/1HGCM82633A004352
- Fixture: fixtures/decodevinvalues-noformat-vin.xml
- Response: `200 application/xml`
- Count: `1`
- SearchCriteria: `VIN(s): 1HGCM82633A004352`
- Proves that default response format is XML.

## 2026-11-08: GetMakesForVehicleType: Car

- GET /GetMakesForVehicleType/car?format=json
- Fixture: fixtures/getmakesforvehicletype-car.json
- Response: `200 application/json`
- Count: `195`
- SearchCriteria: `"Vehicle Type: car"`

## 2026-11-08: GetMakesForVehicleType: Invalid Vehicle Type

- GET /GetMakesForVehicleType/cheese?format=json
- Fixture: fixtures/getmakesforvehicletype-invalid-type.json
- Response: `200 application/json`
- Count: `0`
- Message: `"Response returned successfully"`
- SearchCriteria: `"Vehicle Type: cheese"`
- Results: `[]`

## 2026-11-08: GetModelsForMakeYear: Valid Model Year

Initial attempt made following the format of DecodeVinValues and incorrectly guessed at parameters:

- GET /GetModelsForMakeYear/honda,2015?format=json
- Fixture: fixtures/getmodelsformakeyear-malformed-request.json
- Response: `404 application/json; charset=utf-8`
- Message: `"No HTTP resource was found that matches the request URI 'https://backend-vpic-api.nhtsa.dot.gov/api/vehicles/GetModelsForMakeYear/honda,2015?format=json'."`

Looked up the API on the NHTSA web site and found the correct format:

- GET /GetModelsForMakeYear/make/honda/modelyear/2015?format=json
- Fixture: fixtures/getmodelsformakeyear-valid-make-year.json
- Response: `200 application/json`
- Count: `62`
- SearchCriteria: `"Make:honda | ModelYear:2015"`

## 2026-11-08: GetModelsForMakeYear: Invalid Model Year

- GET /GetModelsForMakeYear/make/honda/modelyear/20x3g?format=json
- Fixture: fixtures/getmodelsformakeyear-invalid-year.json
- Response: `200 application/json`
- Count: `0`
- Message: `"Execution Error"`
- SearchCriteria: `null`
- Results: Misleading `"Error encountered retrieving data: Neither Model Year nor Vehicle Type were supplied"`

## 2026-11-08: DecodeVINValuesBatch: Two VINs

- POST /DecodeVINValuesBatch/
- Form body: `format=json&data=1HGCM82633A004352,2003;5UXWX7C5*BA,2011`
- Fixture: fixtures/decodevinvaluesbatch-two-vins.json
- Response: `200 application/json`
- Count: `2`
- SearchCriteria: `""`
- Results:
  - First VIN:
    - AdditionalErrorText: `""`
    - ErrorCode: `"0"`
    - ErrorText: `"0 - VIN decoded clean. Check Digit (9th position) is correct"`
  - Second VIN:
    - AdditionalErrorText: `""`
    - ErrorCode: `"6"`
    - ErrorText: `"6 - Incomplete VIN"`

## 2026-11-06: CORS verified

- Ran a verification from a Swagger UI scratch page running on localhost (`http://localhost:8080`) and used DevTools > Console:

  ```js
  fetch("https://vpic.nhtsa.dot.gov/api/vehicles/GetMakesForVehicleType/car?format=json").then(r => r.json()).then(d => console.log(d.Count, d.Message))
  ```

- Output with the count and message: `195 'Response returned successfully'`
- Result: Browser cross-origin GET succeeded without a CORS error.
- Action: Need to re-run when published on GitHub Pages in S6.

## 2026-11-06: CORS

- GET /vehicles/GetMakesForVehicleType/car?format=json with Origin: `https://southpaw199.github.io`
- Response header: access-control-allow-origin: *
- Result: browser calls allowed; Swagger UI Try it out should work on GitHub Pages. Confirm in browser during S5.

## 2026-11-06: DecodeVinValues, partial VIN

- GET /vehicles/DecodeVinValues/5UXWX7C5*BA?format=json&modelyear=2011
- Fixture: fixtures/decodevinvalues-partial-vin.json
- Make BMW, Model X3, ModelYear "2011" (string), ErrorCode "6", ErrorText "6 - Incomplete VIN"
- Note: nonzero ErrorCode returned alongside a usable decode; numeric values come back as strings.
