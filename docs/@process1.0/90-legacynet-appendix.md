# Legacynet Appendix

This appendix is for older AO patterns that still matter when supporting existing Legacynet processes.

Use this only when supporting existing Legacynet processes.

## When To Use Legacynet Material

- You are connecting to an existing legacy process.
- A process or tutorial explicitly requires old MU/CU/SU endpoints.
- You need WeaveDrive-on-Legacynet or deprecated dry-run examples.
- You are migrating a process and need to understand old behavior.

For new work, use HyperBEAM mainnet.

## AOS Legacy Flag

Mainnet AOS defaults to HyperBEAM. To create or connect through the legacy path, use `--legacy`:

```sh
aos <legacy-process-id> --wallet ./wallet.json --legacy
```

If connecting directly to an existing legacy process by ID, AOS may detect the legacy network automatically. Keep `--legacy` in examples where the intent must be explicit.

## Old AO Connect Unit Configuration

Older AO Connect examples configured Messenger Unit, Compute Unit, and gateway URLs directly:

```js
import { connect } from "@permaweb/aoconnect";

const { result, results, message, spawn, monitor, unmonitor, dryrun } =
  connect({
    MU_URL: "https://mu.ao-testnet.xyz",
    CU_URL: "https://cu.ao-testnet.xyz",
    GATEWAY_URL: "https://arweave.net",
  });
```

This is a Legacynet pattern. For HyperBEAM mainnet, connect with:

```js
connect({
  MODE: "mainnet",
  URL: "https://<hyperbeam-node>",
  signer,
});
```

## Legacy Spawn Constants

Older spawn examples referenced a scheduler and Legacynet MU authority:

```text
Scheduler:
_GQ33BkPtZrqxA84vM8Zk-N2aO0toNNu_C-l-rawrBA

Legacynet MU authority:
fcoN_xJeisVsPXA-trzVAuIiqO3ydLQxM-L4XbrQKzY
```

These constants are Legacynet-only. For mainnet, use the selected HyperBEAM node authority and the current scheduler flow.

## Dry Run

Dry-run reads belong to the old model. They were used to simulate a message and read a result without saving memory:

```js
import { dryrun } from "@permaweb/aoconnect";

const result = await dryrun({
  process: "PROCESS_ID",
  tags: [{ name: "Action", value: "Balance" }],
});
```

For HyperBEAM, expose state via the [`patch@1.0` device]() and read it through:

```text
GET /<process-id>~process@1.0/compute/<key>
```

## WeaveDrive

The cookbook WeaveDrive snack was written for AO Legacynet. Use it only for existing Legacynet processes unless a current HyperBEAM replacement is available.

Historical setup included:

```sh
aos test-weavedrive \
  --tag-name Extension --tag-value WeaveDrive \
  --tag-name Attestor --tag-value <attestor-address> \
  --tag-name Availability-Type --tag-value Assignments
```

## Older Release Notes

Older release notes are useful for history, not primary onboarding:

- AOS v2.0.7 introduced Hyper AOS support while keeping Legacynet compatibility.
- AOS v2.0.8 added SDK mode and token blueprint state-cache support.
- AOS v2.0.9 improved forward.computer integration and reverted a deduplication feature.

For the current mainnet flow, use [05. AO Connect Mainnet](05-aoconnect-mainnet.md).
