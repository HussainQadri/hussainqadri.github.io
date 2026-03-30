---
title: "FIX Engine: First Update"
date: 2026-03-30
project: fix-engine
---
Date: 2026-03-30
## My thinking

At the moment, the project needs basic parsing so that it can split into tag value pairs, with basic error detection such as verifying body length and checksum

### What Is Implemented

1. `FIXMessage` parses raw SOH-delimited FIX payloads into ordered `tag=value` fields.
2. Validation checks both:
   - `BodyLength (9)` against computed message body size.
   - `CheckSum (10)` using modulo-256 over all bytes before tag `10`.
3. The parser preserves field order and exposes all parsed pairs for downstream logic.
4. Catch2 tests cover valid messages and malformed/failure cases.

### Testing with Catch2

- valid heartbeat / logon / new-order / execution-report style messages
- incorrect checksum
- incorrect body length
- missing checksum
- non-numeric checksum
- duplicate tags (currently treated as invalid by checksum/body-length mismatch)
- truncated message
- empty value field

### Dictionary Status

The project now includes `FIX42.xml` and links `pugixml`.

- `FIXDictionary` currently loads the XML file successfully at startup.
- Tag/message-type lookup methods are declared but not implemented yet.
- Dictionary data is **not** connected to `FIXMessage::validate()` yet.

### Next Step

Integrate the real dictionary model from `FIX42.xml`:

- Parse `<fields>` and `<messages>` into in-memory maps.
- Implement `isValidTag`, `isValidMsgType`, and `getFieldName`.
- Add dictionary-aware validation on top of current checksum/body-length validation.

