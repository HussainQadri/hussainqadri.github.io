---
title: "Dictionary build-up"
date: 2026-03-30
project: fix-engine
---
Date: 2026-03-30

### Parsing the XML 
I am using pugixml to load the xml document I am currently working with. The structure of XML was fairly straight forward, I was able to read the pugixml documentation and read the FIX42.xml dictionary. I used unordered_map to go through the fields and store the attributes, the number was the tag and the value was what that tag means.

The idea is that this should happen only once when on startup, then we should get fast lookups. At the moment, I am working with a local dictionary but the plan is to eventually move to an online dictionary through API calls.
