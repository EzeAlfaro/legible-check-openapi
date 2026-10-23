# legible-check — OpenAPI 3.0.3

A complete OpenAPI 3.0.3 description of a live paid API that charges per call with
x402 (HTTP 402) on Base mainnet. Written as a working sample: every path below is
served today at `https://legible-check.ezealfa.workers.dev`.

## What makes this a useful example

Most OpenAPI documents describe authenticated JSON APIs. A pay-per-call endpoint has
two shapes a buyer has to understand before paying, and both are modelled here:

* **`402` with two transports.** The payment terms are returned in the body *and* in
  a base64 `PAYMENT-REQUIRED` header, because x402 clients differ in which one they
  read. The `PaymentRequired`, `ResourceInfo` and `PaymentOption` schemas capture the
  whole challenge, including the CAIP-2 network, atomic-unit amount as a string,
  `payTo`, and the EIP-712 `extra` block.
* **`200` with a settlement receipt.** The paid response carries a `PAYMENT-RESPONSE`
  header alongside the JSON body, so the receipt is documented as a response header
  rather than smuggled into the schema.

Error documentation is explicit: `400` for a rejected parameter, `404` for a missing
resource, `402` for an unpaid call, `429` for rate limiting, each with an example.

## Checks

```bash
# Schema validity (OpenAPI 3.0 rules, not just YAML)
python -m pip install openapi-spec-validator
python -c "from openapi_spec_validator import validate; from openapi_spec_validator.readers import read_from_filename; s,_=read_from_filename('openapi.yaml'); validate(s); print('valid')"

# Lint with Redocly (the checker most registries use)
npx --yes @redocly/cli@latest lint openapi.yaml
```

Current result: `openapi-spec-validator` passes; Redocly reports **0 errors** and two
style warnings (`operation-4xx-response` on the free docs route and
`no-invalid-media-type-examples` on the deliberately loose `Error` schema, which is
`additionalProperties: true` because the service adds fields per failure).

## Contents

| File | Purpose |
| --- | --- |
| `openapi.yaml` | The specification: 4 paths, 8 schemas, 5 reusable responses |
| `README.md` | This file |

## Coverage

| Path | Method | Price | Notes |
| --- | --- | --- | --- |
| `/` | GET | free | Markdown usage page |
| `/check` | GET | 0.10 USDC | Legibility report for one hashn listing |
| `/check/demo` | GET | free | Same report shape, fixed listing |
| `/x402check` | GET | 0.005 USDC | Preflight of another seller's 402 challenge |
