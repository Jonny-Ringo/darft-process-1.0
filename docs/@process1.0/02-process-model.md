# Process Model

The `process@1.0` device exposes an AO process through HyperBEAM. A process is an autonomous compute object that receives messages, runs handlers, updates internal state, and can send messages or spawn more processes.

## Core Ideas

- A process is the running stateful instance.
- A module is the code used to initialize or execute the process.
- A message is a signed ANS-104 data item addressed to a process.
- A handler matches messages and runs process logic.
- The inbox keeps unhandled messages.
- `ao.id` is the current process ID inside AOS.
- `ao.send` and `Send` send outbound messages.
- `ao.spawn` can create a new process from a module.

## Message Shape

Messages include data, target, sender, tags, signatures, and scheduling metadata. Handlers usually care about:

```lua
msg.From
msg.Target
msg.Data
msg.Tags.Action
msg.Tags.Amount
msg.Tags.Recipient
```

Keep all tag values as strings unless a process explicitly documents otherwise.

## Handlers

A handler has three parts:

1. A name.
2. A matcher.
3. A function that runs when the matcher returns true.

```lua
Handlers.add(
  "Register",
  function(msg)
    return msg.Tags.Action == "Register"
  end,
  function(msg)
    Members = Members or {}
    table.insert(Members, msg.From)

    Send({
      Target = msg.From,
      Data = "Registered"
    })
  end
)
```

The helper matcher is cleaner for common action tags:

```lua
Handlers.add(
  "Ping",
  Handlers.utils.hasMatchingTag("Action", "Ping"),
  function(msg)
    return msg.reply({ Data = "Pong" })
  end
)
```

## Process State

Process memory is local to the process. HyperBEAM reads should expose only the keys a client needs:

```lua
Balances = Balances or {}

Send({
  device = "patch@1.0",
  balances = Balances
})
```

The patched data is readable at:

```text
https://<hyperbeam-node>/<process-id>~process@1.0/compute/balances
```

## Sending Messages

From inside AOS:

```lua
Send({
  Target = ao.id,
  Tags = {
    Action = "Increment"
  },
  Data = "optional body"
})
```

From another process:

```lua
ao.send({
  Target = "<target-process-id>",
  Action = "Notify",
  Data = "hello"
})
```

## Spawning Processes

Spawning requires a module and network-specific scheduling/authority configuration. In new mainnet work, prefer the AO Connect mainnet flow in [05. AO Connect Mainnet](05-aoconnect-mainnet.md).

## `@PROCESS1.0` Mental Model

Think of `process@1.0` as the device boundary where HTTP paths become process operations:

```text
/<process-id>~process@1.0/push
/<process-id>~process@1.0/compute/<key>
/<process-id>~process@1.0/now/<key>
```

Messages still mutate state. HTTP reads should observe state or computed views of state.
