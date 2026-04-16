---
title: "Reading and parsing the XML FIX Dictionary"
date: 2026-04-16
project: fix-engine
---

### Understanding XML dictionary structure

An XML file has the concept of nodes and nodes can be children of other nodes. The 'node' I was looking at was the 'field' node, which contained the mapping between a tag number and what it means. This isn't enough. A FIX string is more complicated than this.

### What a FIX string is actually made of 

A FIX string is made of a header, a message body and a trailer. Every FIX string, has 'required' fields for the header, message body and trailer. For example, the header must contain a field denoting the message type, a `NewOrderSingle` for example. The XML dictionary contains the required fields for a certain message type, continuing with our `NewOrderSingle` example, the XML dictionary would look like this: 

```xml
<message name='NewOrderSingle' msgtype='D' msgcat='app'>
 <field name='ClOrdID' required='Y' />
 <field name='ClientID' required='N' />
 <field name='ExecBroker' required='N' />
 <field name='Account' required='N' />
```

We can see here that the message name is `NewOrderSingle` with associated `msgtype='D'`. The `D` is what will be on the FIX string. Below the first node, there are lines of xml that begin with `<field name = ...`. These are nodes that are the children of `<message>`. The 4 `<field>` lines are describing the possible values that `NewOrderSingle` can take and whether they are required or not. The requirement value is incredibly important and makes the approach to validation *easier*. My previous original thinking was that we move to a typed message model, effectively classes for different message types (which is what I am still doing) but each message type will hold the required fields. This would become messy fast. With dictionary based validation, we can do validation at the parsing step. 

This example was about the message types, the XML also has `<header>` and `<trailer>`, which contain information about what values exist for the header and trailer and whether they are applied or not. Having understood the structure, traversing the XML with `pugixml` is now trivial. `FIXDictionary` contains hashmaps between tags and values, values and tags, as well as for enums...

### Enum business

Some fields in FIX don't just take any value — they take from a fixed set of options. For example, tag `54` (Side) can be `1` for Buy or `2` for Sell. In the XML dictionary, these are represented as `<value>` nodes nested inside a `<field>` node:

```xml
<field number='54' name='Side' type='CHAR'>
 <value enum='1' description='BUY' />
 <value enum='2' description='SELL' />
</field>
```

`FIXDictionary` stores these in a nested hashmap — outer key is the tag number, inner key is the enum value, and the result is the human-readable description. So given a raw FIX tag-value pair like `54=1`, we can resolve it to `Side=BUY`. This is handled by `getEnumDescription()` which looks the description up from the nested hashmap. 
