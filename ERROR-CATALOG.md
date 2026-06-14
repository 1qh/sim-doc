# ERROR-CATALOG

Every error code that crosses a server boundary or surfaces to the user. Typed enum. Per `API-CONVENTIONS.md` error vocabulary + `book/HARD-RULES.md` "Maximum typesafety".

## Server error codes

| Code | HTTP | User message | Recovery |
|---|---|---|---|
| `BAD_INPUT` | 400 | "The input couldn't be processed. Check the highlighted field." | Show field-level errors, focus first invalid field |
| `NOT_FOUND` | 404 | "We couldn't find that." | Suggest related items if applicable, otherwise return to home |
| `RATE_LIMITED` | 429 | "Too many actions. Wait a moment." | Auto-retry after retry-after header |
| `INTERNAL` | 500 | "Something went wrong on our side." | Log + platform error reporter (when enabled), offer retry |
| `SERVICE_UNAVAILABLE` | 503 | "The compute service is temporarily unavailable." | Auto-retry with exponential backoff up to 3 times |

## Client-side error codes

| Code | Surface | User message | Recovery |
|---|---|---|---|
| `CLIENT_PARSE_ERROR` | Asm editor | Inline squiggly on the offending token + line marker | Show error in editor's diagnostics panel |
| `CLIENT_INVALID_KMAP` | K-map input | Inline error on truth table field | Highlight invalid cell, prevent solver run |
| `CLIENT_SIM_DIVERGED` | Datapath / pipeline | Toast "Sim state hash mismatch — reload to recover" | Reload page button in toast |
| `CLIENT_DECODE_FAILED` | Permalink load | Toast "Couldn't decode this permalink" + link to docs | Offer "report broken link" via security disclosure |
| `CLIENT_WEBGPU_UNAVAILABLE` | 3D scene init | Silent fallback to WebGL, no user-visible error | Continue with WebGL renderer |
| `CLIENT_3D_CONTEXT_LOST` | 3D scene mid-runtime | Toast "3D context lost. Reloading scene." | Auto-reload scene state |

## Domain-specific error codes (live inside `packages/sim-engine` + product features)

| Code | Use | User message |
|---|---|---|
| `ASM_LEX_ERROR` | Asm tokenizer failure | "Unexpected character" + position |
| `ASM_SYNTAX_ERROR` | Asm parser failure | "Syntax error" + context |
| `ASM_UNDEFINED_LABEL` | Reference to undefined label | "Label `{name}` not defined" |
| `ASM_DUPLICATE_LABEL` | Two definitions of same label | "Label `{name}` defined twice" |
| `ASM_IMMEDIATE_RANGE` | Immediate out of bounds | "Immediate must be in range {min}..{max}" |
| `ASM_REGISTER_INVALID` | Unknown register name | "Unknown register `{name}`" |
| `ISA_UNSUPPORTED` | Unrecognized opcode | "Instruction `{mnemonic}` not in supported set" |
| `KMAP_INVALID_MINTERMS` | Minterm list has out-of-range or duplicate values | "Minterm out of range for {n}-variable function" |
| `KMAP_TRUTH_TABLE_INCOMPLETE` | Truth table missing rows | "Truth table has {filled}/{required} rows filled" |
| `KMAP_BOOLEAN_EXPR_PARSE` | Boolean expression syntax error | "Couldn't parse Boolean expression" |
| `SIM_DETERMINISM_LEAK` | Internal — frame-N hash mismatch | Throw, log, report to platform error reporter; user sees `CLIENT_SIM_DIVERGED` toast |
| `CODEC_VERSION_UNSUPPORTED` | Snapshot has unknown schema version | Throw, log; user sees `CLIENT_DECODE_FAILED` toast |
| `CODEC_INTEGRITY_FAIL` | Decode integrity check fails on a share fragment | Throw, log; user sees `CLIENT_DECODE_FAILED` toast |

## Error display rules

- Errors quoted exact in development; redacted in production
- No PII in error messages
- Errors never include secrets or internal paths
- Stack traces logged server-side, never sent to client
- User messages domain language, no jargon, no technical noise
- Recovery action always provided where one exists

## Logging

Every error logged with structured fields (per `OBSERVABILITY.md`):

```json
{
  "level": "error",
  "errCode": "BAD_INPUT",
  "op": "assembleProgram",
  "fields": ["source"],
  "duration_ms": 12
}
```

PII never logged. Inputs scrubbed before log emission.

## Caught by

- TypeScript: error codes are a discriminated union, exhaustive switch required at every consumer
- `tools/lint/error-catalog.ts` greps for error code literals in source; every literal must appear in this catalog
- Smoke test per code: trigger condition, assert correct HTTP status + body shape
