# Figma Token Export Script

Run this in the Figma Desktop Bridge (`figma_execute`) or Plugin Console to extract all variables.

```javascript
const rgbaToHex = (r, g, b, a) => {
  const toHex = (v) => Math.round(v * 255).toString(16).padStart(2, '0');
  const hex = `#${toHex(r)}${toHex(g)}${toHex(b)}`;
  if (a !== undefined && a < 1) {
    return `rgba(${Math.round(r*255)}, ${Math.round(g*255)}, ${Math.round(b*255)}, ${parseFloat(a.toFixed(2))})`;
  }
  return hex;
};

const toCssName = (prefix, figmaName) => {
  return `--${prefix}-` + figmaName
    .toLowerCase()
    .replace(/\//g, '-')
    .replace(/\s+/g, '-')
    .replace(/[()]/g, '')
    .replace(/-+/g, '-');
};

async function exportTokens() {
  const collections = await figma.variables.getLocalVariableCollectionsAsync();
  const output = {
    collections: [],
    meta: {
      exportedAt: new Date().toISOString(),
      fileKey: figma.fileKey
    }
  };

  for (const col of collections) {
    const colData = {
      name: col.name,
      id: col.id,
      modes: col.modes.map(m => ({ id: m.modeId, name: m.name })),
      variables: []
    };

    // Process in batches of 8 to avoid timeouts
    const batches = [];
    for (let i = 0; i < col.variableIds.length; i += 8) {
      batches.push(col.variableIds.slice(i, i + 8));
    }

    for (const batch of batches) {
      for (const varId of batch) {
        const v = await figma.variables.getVariableByIdAsync(varId);
        if (!v) continue;

        const varData = {
          name: v.name,
          id: v.id,
          type: v.resolvedType,
          values: {}
        };

        for (const mode of col.modes) {
          const val = v.valuesByMode[mode.modeId];
          if (val && val.type === 'VARIABLE_ALIAS') {
            const ref = await figma.variables.getVariableByIdAsync(val.id);
            varData.values[mode.name] = {
              type: 'alias',
              aliasTo: ref ? ref.name : val.id
            };
          } else if (val && typeof val === 'object' && 'r' in val) {
            varData.values[mode.name] = {
              type: 'color',
              value: rgbaToHex(val.r, val.g, val.b, val.a)
            };
          } else {
            varData.values[mode.name] = {
              type: v.resolvedType === 'FLOAT' ? 'number' : typeof val,
              value: val
            };
          }
        }

        colData.variables.push(varData);
      }
    }

    output.collections.push(colData);
  }

  return output;
}

return await exportTokens();
```

## Important Notes

- **Async is mandatory**: All `getVariableBy*` calls must use async versions
- **Batch size**: Process max 8 variables per loop iteration to avoid Bridge timeouts
- **Alias resolution**: The script resolves one level of aliasing. For full chain resolution, walk aliases recursively
- **Color format**: RGBA objects from Figma are `{r, g, b, a}` with 0-1 float values, not 0-255
- **String variables**: Font families come through as plain strings
- **FLOAT variables**: Sizes and weights come through as numbers

## Output Format

```json
{
  "collections": [
    {
      "name": "Primitives",
      "modes": [{ "id": "1:0", "name": "Dark" }],
      "variables": [
        {
          "name": "Size/16",
          "type": "FLOAT",
          "values": { "Dark": { "type": "number", "value": 16 } }
        },
        {
          "name": "Color/Neutral/500",
          "type": "COLOR",
          "values": { "Dark": { "type": "color", "value": "#9da0a9" } }
        }
      ]
    },
    {
      "name": "Semantic Tokens",
      "modes": [
        { "id": "1:2", "name": "Desktop" },
        { "id": "355:0", "name": "Tablet" },
        { "id": "4200:0", "name": "Mobile" }
      ],
      "variables": [
        {
          "name": "Typography/Size/XXL",
          "type": "FLOAT",
          "values": {
            "Desktop": { "type": "alias", "aliasTo": "Size/60" },
            "Tablet": { "type": "alias", "aliasTo": "Size/60" },
            "Mobile": { "type": "alias", "aliasTo": "Size/48" }
          }
        }
      ]
    }
  ]
}
```
