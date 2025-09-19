# Mellowtel Bundled

This repository contains a pre-bundled version of Mellowtel that can be used in browser extensions without requiring bundlers like Webpack or Vite.

## Quick Start

The `mellowtel.js` file in this repository is ready to use in your browser extension. Simply copy it to your extension directory and follow the integration steps below.

## Project Setup

Start with a basic browser extension structure:

```
your-extension/
├── icons/
├── popup/
│   ├── popup.html
│   └── popup.js
├── background.js           # Your service worker
├── content_script.js
├── manifest.json
└── mellowtel.js           # Copy from this repository
```

## Manifest Configuration

Configure your `manifest.json` to include the Mellowtel bundle in content scripts:

```json
{
  "manifest_version": 3,
  "name": "Your Extension Name",
  "version": "1.0",
  "background": {
    "service_worker": "background.js",
    "type": "module"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["mellowtel.js", "content_script.js"]
    }
  ],
  "permissions": [
    "storage",
    "declarativeNetRequestWithHostAccess"
  ],
  "host_permissions": [
    "<all_urls>"
  ]
}
```

**Important**: 
- The `"type": "module"` field enables ES module imports in your service worker.
- In the content scripts array, `mellowtel.js` must be loaded **before** `content_script.js` so that the Mellowtel global variable is available when your content script runs.

## Usage in Your Extension

### Service Worker (background.js)

Import the bundled file in your service worker:

```javascript
// background.js
import './mellowtel.js';

(async () => {
    const mellowtel = new Mellowtel("<configuration_key>"); // Replace with your configuration key
    await mellowtel.initBackground();
})();
```

### Content Scripts

Mellowtel is available as a global variable (no import needed, as it's loaded via the manifest):

```javascript
// content_script.js
// Mellowtel is available as a global variable from mellowtel.js

(async () => {
  const mellowtel = new Mellowtel("<configuration_key>"); // Replace with your configuration key
  await mellowtel.initContentScript();
})();
```

## Testing Your Extension

To test your extension:

1. Go to `chrome://extensions/` in Chrome (or equivalent in other browsers)
2. Enable "Developer mode"
3. Click "Load unpacked" and select your extension folder

## Generating Your Own Bundle

If you prefer to generate the bundle yourself, follow these steps:

### Step 1: Create a Temporary Bundling Directory

Create a temporary directory to install Mellowtel and generate the bundle:

```bash
mkdir mellowtel-bundling
cd mellowtel-bundling
npm init -y
npm install mellowtel
```

This creates a separate folder with the Mellowtel package that you'll use only for bundling.

### Step 2: Bundle Mellowtel with esbuild

From inside the `mellowtel-bundling` directory, create a single bundle that works in both service workers and content scripts:

```bash
npx esbuild node_modules/mellowtel/dist/index.js --bundle --format=iife --global-name=Mellowtel --footer:js="globalThis.Mellowtel = Mellowtel.default" --platform=browser --outfile=mellowtel.js --minify --legal-comments=none
```

This command:
- Takes the Mellowtel source from `node_modules/mellowtel/dist/index.js`
- Bundles all dependencies into a single file
- Uses IIFE format with a global name for universal compatibility
- Adds a footer to make the default export available globally
- Optimizes for browser environment and minifies the output
- Creates `mellowtel.js` in the bundling directory

### Step 3: Copy the Bundle to Your Extension

After running the esbuild command, copy the generated file to your extension directory:

```bash
cp mellowtel.js ../your-extension/
```

Now you can delete the temporary bundling directory since you only needed the output file:

```bash
cd ..
rm -rf mellowtel-bundling
```

Your extension folder now contains the bundled Mellowtel file ready to use.

## Benefits

This approach gives you the benefits of using Mellowtel without the complexity of setting up a full build system like Webpack or Vite. The pre-bundled file works universally in both service workers and content scripts.