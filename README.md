# fastapi-vite

Integration for FastAPI and Vite JS

## what?

This package is designed to make working with javascript assets easier.

fastapi-vite enables the jinja filters required to render asset URLs to jinja templates

Inspired by `django-vite` @ [https://github.com/MrBin99/django-vite]

## installation

Install using pip

```shell
pip install fastapi-vite
```

## Usage

Configure Jinja templating for FastAPI

``` python
import fastapi_vite

templates = Jinja2Templates(directory='templates')
templates.env.globals['vite_hmr_client'] = fastapi_vite.vite_hmr_client
templates.env.globals['vite_asset'] = fastapi_vite.vite_asset

```

### Configure Vite

Here is an example used to test this plugin

``` javascript
// vite.config.js
import {defineConfig} from 'vite'
import reactRefresh from '@vitejs/plugin-react-refresh'
import path from "path";

const Dotenv = require("dotenv");

Dotenv.config({path: path.join(__dirname, ".env")});

const STATIC_URL = process.env.STATIC_URL;
// https://vitejs.dev/config/
export default defineConfig({
  base: `${STATIC_URL}`,
  clearScreen: false,
  plugins: [
    reactRefresh(),  // for react
    vue()  // for vue
  ],
  resolve: {
    alias: {
      // allow use of @ in import statements
      "@": fileURLToPath(new URL("./src", import.meta.url)),
    },
  },
  server: {
    // server setting sto work around access-control-allow-origin problems in dev
    fs: {
      strict: false,
      allow: [
        // search up for workspace root
        searchForWorkspaceRoot(process.cwd()),
        // your custom rules
        "../static/assets",
      ],
    },
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, PATCH, OPTIONS",
      "Access-Control-Allow-Headers": "X-Requested-With, content-type, Authorization",
    },
  },
  build: {
    target: "esnext",
    outDir: "./static/",
    emptyOutDir: true,
    assetsDir: "",
    manifest: true,
    rollupOptions: {
      input: {
        main: resolve("./src/main.js"),
      },
      output: {
        chunkFileNames: undefined,
      },
    }
  },

  root: ".", // You can change the root path as you wish

})

```

### Configure Static Assets

Configure with `.env` file with these settings

| Option           | Default     | Explanation            |
|------------------|-------------|------------------------|
| VITE_SERVER_HOST | "localhost" | Vite server host       |
| SERVER_PROTOCOL  | "http"      | Vite server protocol   |
| SERVER_PORT      | 5173        | Vite server port       |
| ASSETS_PATH      | "static/"   |                        |
| MANIFEST_PATH    | -           |                        |
| STATIC_PATH      | "static/"   |                        |
| HOT_RELOAD       | -           | Autodetect from DEBUG  |
| IS_REACT         | False       |                        |


### Configure Templates

\*render_vite_hmr no-op when in production.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <!--IE compatibility-->
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, maximum-scale=1.0"
    />
  </head>

  <body>
    <div id="app"></div>
    {{ vite_hmr_client() }}
    {{ vite_asset('src/main.js') }}
  </body>
</html>
```
