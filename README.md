# vPIC Vehicle API (Unofficial)

An unofficial OpenAPI 3.0.3 description of three operations in NHTSA's Product Information Catalog Vehicle Listing (vPIC) Vehicle API. The service decodes and retrieves automotive and manufacturer information from vPIC.

**Disclaimer**: This is an unofficial OpenAPI description. It is not affiliated with or endorsed by [NHTSA](https://vpic.nhtsa.dot.gov/api/).

## Links

- [Live Swagger UI](https://southpaw199.github.io/vpic-openapi/)
- [Verification](VERIFICATION.md)
- [Specification](docs/openapi.yaml)
- [NHTSA vPIC API](https://vpic.nhtsa.dot.gov/api/vehicles)

## Why

NHTSA publishes endpoint docs but no OpenAPI definition. My hope is that by publishing this specification, it will increase access to the information.

## Scope

This specification covers only three of vPIC's operations:

- `DecodeVinValues`
- `DecodeVINValuesBatch`
- `GetMakesForVehicleType`

For a full list of available operations, see [NHTSA Vehicle API](https://vpic.nhtsa.dot.gov/api/).

## Method

1. Requested every operation with `curl` in a `bash` shell.
2. Saved the fixtures.
3. Logged the behaviors in [VERIFICATION.md](VERIFICATION.md).
4. Wrote the specification from the fixtures.
5. Copied examples from the fixtures with `jq`.
6. Linted with Redocly: `npx @redocly/cli lint docs/openapi.yaml` (recommended ruleset): valid, no warnings.
7. Tested with Swagger UI Try it out.

## Key findings

- XML is the default format. Add `format=json` to get JSON.
- `vehicleType` matches any part of a vehicle type name. `passenger` returns makes for both Passenger Car and MPV, with makes that belong to both types listed twice.
- Bad input, such as an invalid VIN or unknown vehicle type, returns `HTTP 200`. Check `ErrorCode`, `ErrorText` in each result. For example, `12345` decodes to Make "SHERMAN + REILLY" with `ErrorCode: 6,8,11` because the first three characters are read as a manufacturer code (WMI). For makes, `Count: 0` and an empty `Results` mean nothing matched.
- Missing or wrong path returns a `404` with a `{"message": ...}` body and exposes the backend hostname.

## Limitations

- `GetModelsForMakeYear` is not included.
- CSV is supported by vPIC, but not described in this specification.
- The effect of `modelyear` on `DecodeVinValues` has not been tested.
- Field-level descriptions are limited.
- Three description claims (the 9th digit is optional, up to 50 VINs per batch, model year is optional in a batch) come from NHTSA's API page and were not tested.

## AI assistance

I ran the requests, captured the fixtures, and wrote the specification, verification log, and README.

Claude reviewed each pass against the fixtures and lint output, explained OpenAPI syntax, and ran checks against the live API.

## License

Released under an [MIT license](https://github.com/southpaw199/vpic-openapi/blob/main/LICENSE).

The vPIC API and its data belong to NHTSA.