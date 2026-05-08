---
title: "Initial session state"
date: 2026-05-08
project: fix-engine
---

### Moving into the session layer

The last update ended with the idea that the engine should move from message construction into session state. I did not want to jump straight into TCP yet because sockets would make it harder to see whether the actual FIX session rules were correct. The useful step before networking is a session object that can build outbound admin messages and react to incoming messages just to get something working.

I added a `FIXSession` class for this. It owns a small `SessionConfig`, which contains the FIX version, sender comp ID, target comp ID, and heartbeat interval. It also tracks the current session state, the next outgoing sequence number, and the next incoming sequence number expected from the counterparty.

The states are deliberately simple for now:

- disconnected
- logon sent
- active
- logout sent

This is not the complete FIX session model yet, but it is enough to start enforcing the basic lifecycle.

### Outbound admin messages

The session can now create the main admin messages that will eventually be sent over the network:

- `Logon`
- `Heartbeat`
- `TestRequest`
- `Logout`
- `ResendRequest`

Each outbound message gets the standard session header fields using `addField()` from `FIXMessage`:

```text
8=FIX.4.2
35=<message type>
49=<sender comp ID>
56=<target comp ID>
34=<next outgoing sequence number>
52=<current UTC sending time>
```

Then the message-specific fields are added. For example, a Logon gets `98=0` for no encryption and `108=<heartbeat interval>`. A TestRequest gets `112=<test request ID>`. A ResendRequest gets `7=<begin seq no>` and `16=<end seq no>`.

After the fields are added, the message is serialized so that `BodyLength(9)` and `CheckSum(10)` are calculated correctly. The outgoing sequence number is then incremented.

### Incoming messages and sequence numbers

The main method is `onIncoming()`. It takes a parsed `FIXMessage` and returns any messages that should be sent back.

Before doing anything session-specific, it checks three things:

1. The message passes wire validation.
2. The message passes dictionary validation.
3. The counterparty fields are in the expected direction.

For example, if my session is configured as `BUY -> SELL`, then an incoming message should be from `SELL -> BUY`. That means tag `49` should be `SELL` and tag `56` should be `BUY`.

After that, the session checks tag `34`, the message sequence number.

If the incoming sequence number is exactly what the session expected, the message is processed and the expected incoming sequence number is incremented.

If the incoming sequence number is too high, the session has detected a gap. For example, if it expected `34=1` but received `34=3`, then messages 1 and 2 are missing. In that case, the session returns a `ResendRequest`:

```text
35=2
7=1
16=2
```

That means "resend messages 1 through 2".

At the moment, low sequence numbers are ignored. That is not the final behaviour, but it is enough for this first pass. Later this needs to distinguish duplicates, possible duplicates, and genuinely invalid low sequence numbers.

### Basic admin reactions

The first session behaviours are now in place:

- A valid incoming Logon moves the session into the active state.
- A TestRequest returns a Heartbeat with the same `112` value.
- A Logout returns a Logout acknowledgement if one has not already been sent.
- A sequence gap returns a ResendRequest.

This is still not a full session layer. There are no heartbeat timers yet, no persistence, no replay of stored messages, no SequenceReset handling, and no TCP transport. But the important part is that the session logic is now separate from networking. When transport is added, it should only need to frame bytes, parse messages, pass them into `FIXSession`, and write back any serialized responses.
