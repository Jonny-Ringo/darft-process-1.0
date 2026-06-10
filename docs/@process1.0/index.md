# @PROCESS1.0 Device

This section covers the HyperBEAM `process@1.0` device surface.

Use it to spawn or connect to AO processes, send messages, expose process state over HTTP, and migrate older Legacynet code into the HyperBEAM model.

## Default Path

HyperBEAM is the primary path for new `@PROCESS1.0` work.

The important URL shape is:

```text
https://<hyperbeam-node>/<process-id>~process@1.0/<operation>
```

Common operations:

- `push`: send a signed message into a process.
- `compute/<key>`: read the last exposed value for a patched key.
- `now/<key>`: read from the most current process state path.
- `now/~lua@5.3a&module=<tx-id>/<function>/serialize~json@1.0`: run a dynamic Lua read over process state.

## Node Selection

Examples in this section may use `https://push.forward.computer`. Treat it only as an example node.

For real use, select a current HyperBEAM node from `https://lunat.arweave.net` or another marketplace/listing source. Use the selected node's URL for `--url`, `URL`, and read endpoints. Use that node's advertised authority address when a workflow asks for `authority` or `HYPERBEAM_AUTHORITY`.

## Section Order

1. Start with the `process@1.0` and processes intro.
2. Learn the process/message/handler model.
3. Use state exposure and dynamic reads for fast reads.
4. Build practical patterns on top.
5. Use AO Connect mainnet APIs for JavaScript clients.
6. Keep AOS/Lua basics close by.
7. Use the migration guide for older processes.
8. Keep Legacynet-only material in the final appendix.

## Scope

This section focuses on the English technical core for `process@1.0`: process creation, messages, state exposure, reads, AO Connect, and migration.

Legacynet-specific items are separate from the main flow. They live in [90. Legacynet Appendix](90-legacynet-appendix.md).
