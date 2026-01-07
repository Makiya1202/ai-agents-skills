---
name: figma
description: Integrate with Figma API for design automation. Use when extracting design tokens, generating code from Figma, syncing design systems, or building Figma plugins. Triggers on Figma API, design tokens, Figma plugin, design-to-code, Figma export.
---

# Figma API Integration

Extract design data and automate design workflows with Figma's API.

## Quick Start

### Authentication
```typescript
const FIGMA_TOKEN = process.env.FIGMA_TOKEN;

const headers = {
  'X-Figma-Token': FIGMA_TOKEN
};

// Get file
const response = await fetch(
  `https://api.figma.com/v1/files/${FILE_KEY}`,
  { headers }
);
```

### File Key
Extract from Figma URL: `figma.com/file/FILE_KEY/...`

## Core API Endpoints

### Get File
```typescript
// Full file
GET https://api.figma.com/v1/files/:file_key

// Specific nodes
GET https://api.figma.com/v1/files/:file_key/nodes?ids=1:2,1:3

// With geometry
GET https://api.figma.com/v1/files/:file_key?geometry=paths
```

### Export Images
```typescript
// Export nodes as images
GET https://api.figma.com/v1/images/:file_key?ids=1:2,1:3&format=png&scale=2

// Response
{
  "images": {
    "1:2": "https://s3.amazonaws.com/...",
    "1:3": "https://s3.amazonaws.com/..."
  }
}
```

### Get Styles
```typescript
GET https://api.figma.com/v1/files/:file_key/styles

// Published styles
GET https://api.figma.com/v1/teams/:team_id/styles
```

## Design Token Extraction

### Extract Colors
```typescript
async function extractColors(fileKey: string) {
  const file = await fetch(
    `https://api.figma.com/v1/files/${fileKey}/styles`,
    { headers }
  ).then(r => r.json());
  
  const colors = {};
  
  for (const style of file.meta.styles) {
    if (style.style_type === 'FILL') {
      const nodeData = await getStyleNode(fileKey, style.node_id);
      const fill = nodeData.fills[0];
      
      if (fill.type === 'SOLID') {
        colors[style.name] = rgbaToHex(fill.color);
      }
    }
  }
  
  return colors;
}

function rgbaToHex({ r, g, b, a = 1 }) {
  const toHex = (n) => Math.round(n * 255).toString(16).padStart(2, '0');
  return `#${toHex(r)}${toHex(g)}${toHex(b)}${a < 1 ? toHex(a) : ''}`;
}
```

### Generate CSS Variables
```typescript
function generateCSSVariables(tokens) {
  let css = ':root {\n';
  
  for (const [name, value] of Object.entries(tokens.colors)) {
    const cssName = name.toLowerCase().replace(/\s+/g, '-');
    css += `  --color-${cssName}: ${value};\n`;
  }
  
  css += '}\n';
  return css;
}
```

## Webhooks

### Setup Webhook
```typescript
POST https://api.figma.com/v2/webhooks

{
  "event_type": "FILE_UPDATE",
  "team_id": "123456",
  "endpoint": "https://your-server.com/figma-webhook",
  "passcode": "your-secret-passcode"
}
```

### Handle Webhook
```typescript
app.post('/figma-webhook', (req, res) => {
  const { passcode } = req.body;
  
  if (passcode !== process.env.FIGMA_WEBHOOK_SECRET) {
    return res.status(401).send('Unauthorized');
  }
  
  const { event_type, file_key } = req.body;
  
  if (event_type === 'FILE_UPDATE') {
    syncDesignTokens(file_key);
  }
  
  res.status(200).send('OK');
});
```

## Resources

- **Figma API Docs**: https://www.figma.com/developers/api
