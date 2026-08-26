---
name: hoc-modules-imports
description: "Conventions for module imports. Group imports at the top of the file, ordered from farthest to nearest to application development."
---

# Modules: Imports

Conventions related to module imports.

## Group imports at the top of the file

- Group all imports together at the top of the file.
- Do not place assignment statements between imports.

## Import order

- Order imports "from farthest to nearest to application development".
- The order is as follows.

1. JavaScript native modules
2. third party dependency modules
3. ORT modules
4. application modules
5. constants

```javascript
// JavaScript native modules
import fs from 'node:fs'

// third party dependency modules
import express from 'express'

// ORT modules
import BaseClass from '@openreach/base'

// application modules
import ColorNormalizer from '../lib/ColorNormalizer.js'

// constants
import COLOR from '../constants/COLOR.js'
```
