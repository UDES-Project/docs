# Web Extension

> UMES Library (@umes/web-ext-library): v1.0.7
> React: v18
> Vite: v6

### Create a Vite project

```bash
npm create vite@latest
√ Project name: umes-client
√ Select a framework: » React
√ Select a variant: » TypeScript
```

### Install dependencies

```bash
cd umes-client
npm install
```

### Add minifest file

Create a `manifest.json` in the `public` folder

```json
{
    "manifest_version": 2,
    "name": "UMES Client",
    "version": "1.0",
    "background": {
        "scripts": [
            "background.js"
        ],
        "persistent": false
    },
    "content_scripts": [
        {
            "matches": [
                "{app_url}"
            ],
            "js": [
                "content.js"
            ],
            "run_at": "document_end"
        }
    ],
    "browser_action": {
        "default_popup": "popup.html"
    },
    "web_accessible_resources": [
        "script.js"
    ]
}
```

### Build configuration

Edit the configuration in the `vite.config.ts` file

```js
export default defineConfig({
    plugins: [react()],
    build: {
        minify: false,
        rollupOptions: {
            input: {
                content: 'src/ext/content.js',
                script: 'src/ext/script.js',
                background: 'src/ext/background.js',
                main: 'src/popup/main.tsx',
            },
            output: {
                entryFileNames: '[name].js',
                chunkFileNames: '[name].js',
                assetFileNames: '[name].[ext]',
            },
        }
    }
})
```

### Create the popup page

 - Create a `popup` folder in `src/`
 - Add all files of `src/` in it (without `vite-env.d.ts`)
 - Move the `/index.html` to the `public` folder and rename it `popup.html`
 - Now you can edit the `popup` folder as you want

### Use the UMES Library

Install the npm package

```bash
npm i @umes/web-ext-library
```

Create a the folder `ext` in `src/`, and create 3 files, `background.js`, `content.js`, `script.js`

> TypeScript is not used because the `browser` variable is not typed

Here are exemples of contents:

`src/ext/background.js`
```js
import { UMES_Background } from "@umes/web-ext-library"

var UMES = new UMES_Background("Extension Name", "icons/iconfull.png")
```

`src/ext/content.js`
```js
import { UMES_ContentScript } from "@umes/web-ext-library"

(async () => {
    var UMES = new UMES_ContentScript("http://umes.ddns.net/api", true)

    UMES.setMessageContainer("{message_container_css_selector}", "{inner_message_css_selector}", (message) => {
        var content = message.textContent
        
        if (UMES.isUMESMessage(content)) {
            UMES.decryptMessage(content, (decrypted) => {
                message.textContent = decrypted
            })
        }
    })

    UMES.injectScript("script.js", "body")
})()
```

`src/ext/script.js`
```js
import { UMES_Script } from "@umes/web-ext-library"

var UMES = new UMES_Script()

WebSocket.prototype._send = WebSocket.prototype.send

WebSocket.prototype.send = function(payload) {
    payload = JSON.parse(payload)

    if (payload.event == "message" && payload.data.content) {

        UMES.encryptMessage(payload.data.content, (public_id, key) => {
            payload.data.content = `[UMES]${public_id}:${key}`
            this._send(JSON.stringify(payload))
        })

    } else {
        this._send(JSON.stringify(payload))
    }
}
```

### Build and test

```
npm run build
```

The go to the `about:addons` in Firefox and go to `Debug Add-ons` (`about:debugging#/runtime/this-firefox`), the load temporary add-on, and selected the `manifest.json` in the `dist` folder