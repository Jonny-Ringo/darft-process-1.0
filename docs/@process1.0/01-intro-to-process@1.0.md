# Intro to `process@1.0` & Processes

AO has many devices. Among them, the `process@1.0` device is the one you use to create and interact with "smart contracts" called processes.

A process is a deterministic log of state transitions stored on Arweave. Messages add inputs to that log, the process evaluates those messages, and HyperBEAM makes selected process data available through endpoints.

Use this page to create a process, load a small counter, send it a message, and read exposed state through the `process@1.0` device.

## Process Model

- A process is the running stateful instance.
- A module is the code used to initialize or execute the process.
- A message is a signed ANS-104 data item addressed to a process.
- A handler matches messages and runs process logic.
- The inbox keeps unhandled messages.
- `ao.id` is the current process ID inside AOS.
- `Send` sends outbound messages.
- `ao.spawn` can create a new process from a module.

## Message Shape

Messages include data, target, sender, tags, signatures, and scheduling metadata. The core fields a handler usually reads are:

```lua
msg.From
msg.Target
msg.Data
```

`msg.Data` can be empty, but it is the right place for larger payloads. Tags can carry routing and metadata such as `Action`, `Amount`, or `Recipient`.

ANS-104 tag limits:

- Up to 128 tags per message.
- Tag key/name: up to 1024 bytes.
- Tag value: up to 3072 bytes.

Keep tag values as strings unless a process explicitly documents otherwise.

## Process Authorities

For process-to-process messages to be accepted, the recipient process must trust the authority used by the incoming message. In practice, processes that talk to each other should use the same HyperBEAM authority or compatible authority sets. If the recipient does not have the authority, it can reject the message.

# Using AOS

## Prerequisites

- Node.js 20 or newer.
- A terminal.
- An Arweave wallet keyfile, or let AOS create one.
- A HyperBEAM node URL selected from `https://lunar.arweave.net` or another current marketplace/list.

## Install AOS

```sh
npm i -g https://get_ao.arweave.net
```

## Create a Process

```sh
HYPERBEAM_URL=https://push.forward.computer
aos process_name --url "$HYPERBEAM_URL"
```

Example:

```sh
aos counter --url https://push.forward.computer
```

Replace `process_name` with a friendly local name for the process, such as `counter`. Reuse the same command to re-enter the process console after exiting. If you omit the node URL, the current AOS mainnet release defaults to a HyperBEAM node like `push.forward.computer`. Passing a URL keeps the selected node explicit as the executor of the process.

To use a specific wallet:

```sh
HYPERBEAM_URL=https://push.forward.computer
aos process_name --wallet ./wallet.json --url "$HYPERBEAM_URL"
```

## Create A Counter

To paste multiline Lua into the AOS prompt, open the inline editor first:

```text
.editor
```

Paste the counter code below into the editor, then type this on its own line to exit the editor and execute the code:

```text
.done
```

Alternatively, save the code below to an `example.lua` file and load it with:

```text
.load path/to/example.lua
```

Note: The [`patch@1.0` device](02-state-and-reads.md#patch-device) pushes selected process data to HyperBEAM endpoints so clients can discover and fetch it. In this example, the `counter`, `lastupdate`, and `updatedby` keys become readable through the process endpoint.

The code below registers an `Increment` handler. A handler has a name, a matcher, and a function that runs when the matcher succeeds.

```lua
Counter = Counter or 0

Handlers.add(
  "Increment",
  Handlers.utils.hasMatchingTag("Action", "Increment"),
  function(msg)
    Counter = Counter + 1

    Send({
      device = "patch@1.0",
      counter = tostring(Counter),
      lastupdate = tostring(os.time()),
      updatedby = msg.From
    })

    return msg.reply({
      Status = "OK",
      Counter = tostring(Counter)
    })
  end
)
```

Send a message to your own process:

```lua
Send({ Target = ao.id, Tags = { Action = "Increment" } })
```

On mainnet, a self-send is an outbound message. It can be processed asynchronously and may not increment the counter before your next read. For a repeatable tutorial test, send the `Action=Increment` message from an external client such as AO Connect, then read the patched key.

## Read State Through `process@1.0`

Use your selected node and process ID:

```sh
HYPERBEAM_URL=https://push.forward.computer
PROCESS_ID=YOUR_PROCESS_ID
curl "$HYPERBEAM_URL/$PROCESS_ID~process@1.0/compute/counter"
```

If you are using AO Connect, send the increment as a normal external message and pass the returned slot into `ao.result`:

```js
const slot = await ao.message({
  process: processId,
  tags: [{ name: "Action", value: "Increment" }],
});

const result = await ao.result({
  process: processId,
  slot,
});
```

The response is the patched `counter` value. This is the core HyperBEAM read pattern: mutate process state through messages, then expose selected read state over HTTP.

## Initial State Sync

Patch important read keys once when the process starts:

```lua
Counter = Counter or 0
InitialSync = InitialSync or "INCOMPLETE"

if InitialSync == "INCOMPLETE" then
  Send({
    device = "patch@1.0",
    counter = tostring(Counter)
  })

  InitialSync = "COMPLETE"
end
```

## Naming Rules

- Prefer lowercase patch keys: `counter`, `balances`, `totalsupply`.
- Avoid reserved path words as keys: `now`, `compute`, `state`, `info`, `test`.
- Prefer dash-separated tag names: `Example-Tag`, `Transfer-Id`.
- Do not rely on mixed-case tag normalization. HyperBEAM can normalize tag casing in ways that break older handlers.

## Local HyperBEAM Node

For local development, start HyperBEAM with the profile your environment requires, then point AOS at it:

```sh
aos --url http://localhost:8734 myLocalProcess
```

The upstream cookbook commonly references a `genesis_wasm` profile for local AOS execution. Treat the exact local boot command as environment-specific and keep the selected node URL explicit.
