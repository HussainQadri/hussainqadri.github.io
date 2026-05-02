---
title: "Message construction and the path to sessions"
date: 2026-05-02
project: fix-engine
---

### Moving from parsing to construction

Up to this point, the engine has mostly worked in one direction: take a raw FIX string, split it into ordered tag-value pairs, and then validate the structural parts of the message. That gave me the parser, checksum validation, body length validation, dictionary-backed field lookup, and typed wrappers for the core admin messages.

The next missing piece was being able to go the other way. A FIX engine should not only read messages, it should also be able to build them. This is where `FIXMessage::serialize()` comes in.

The goal of serialization is to turn the internal field representation back into a valid FIX wire message:

```text
8=FIX.4.2<SOH>9=...<SOH>35=...<SOH>49=...<SOH>56=...<SOH>10=...<SOH>
```

The important part is that tags `9` and `10` cannot just be copied from the current object. `BodyLength(9)` and `CheckSum(10)` are derived from the message itself, so they have to be recalculated whenever a message is serialized.

### BodyLength and CheckSum

`BodyLength` is the number of bytes after the complete `9=<value><SOH>` field and before the checksum field. It includes the tag digits, equals signs, values, and SOH separators for the body fields.

For example:

```text
35=0<SOH>
```

has length 5:

```text
3 + 5 + = + 0 + SOH
```

`CheckSum` is different. It is calculated by summing every byte from the start of the message up to, but not including, tag `10`, then taking the result modulo 256. FIX also expects it to be formatted as exactly three digits, so a checksum of `8` becomes `008`.

The serialization flow now looks like this:

1. Build the body string from all fields except `8`, `9`, and `10`.
2. Add tag `8`.
3. Add a freshly calculated tag `9`.
4. Append the body.
5. Calculate the checksum over everything built so far.
6. Add tag `10` as a three-digit value.

This means a message can be parsed, modified later, and serialized with fresh structural fields instead of stale ones.

### Building messages from scratch

I also added a default constructor and a public `addField()` method. This means `FIXMessage` is no longer only a parser around an existing raw string. It can also be used as a small message builder:

```cpp
FIXMessage msg;
msg.addField("8", "FIX.4.2");
msg.addField("35", "0");
msg.addField("49", "SENDER");
msg.addField("56", "TARGET");
msg.addField("34", "2");
msg.addField("52", "20260311-12:00:00.000");

std::string wire = msg.serialize();
```

The distinction between `addField()` and the internal serialization helper matters. `addField()` updates the actual message state: the ordered field vector and lookup map. The helper used inside `serialize()` only appends `tag=value<SOH>` text to an output string.

This gives the engine a basic round-trip path:

```text
raw FIX string -> FIXMessage -> serialized FIX string -> FIXMessage -> validate()
```

That is a useful foundation because the session layer will need to create outbound messages constantly. Unfortunately, this is causing the `FIXMessage` class to be bloated, if we have `serialize()` in this class then I think this class needs be broken down further. We should have a *factory* type class that can simply build valid FIX messages.

### Next: session state

The next step is the session layer. I am not going straight to TCP sockets yet. The first version should be session state and sequence numbers without networking.

The initial session model should track:

- sender and target comp IDs
- outgoing sequence number
- expected incoming sequence number
- disconnected, logon-sent, active, and logout-sent states

The rules can start small:

- outbound admin messages get `BeginString`, `MsgType`, `SenderCompID`, `TargetCompID`, `MsgSeqNum`, and `SendingTime`
- creating an outbound message increments the outgoing sequence number
- receiving a message checks whether tag `34` matches the expected incoming sequence number
- valid incoming messages increment the expected incoming sequence number
- a valid incoming Logon moves the session into the active state

After that, the harder parts can be layered on: heartbeat timers, test requests, resend requests, gap fills, persistence, and finally TCP transport.
