# Electron Project
- Creating an ElectronJS app that uses SQLite as a database, incorporates React with React DevTools, and uses Vite instead of Webpack
  involves several steps. Below is a step-by-step guide to help you set up your project:

---

## **1. Initialize Your Project**

Create a new directory for your project and initialize it with npm:

```bash
mkdir electron-react-vite-sqlite-app
cd electron-react-vite-sqlite-app
npm init -y
```

## **2. Install Dependencies**

Install the necessary packages:

```bash
# Electron as a dev dependency
npm install --save-dev electron

# React and ReactDOM
npm install react react-dom

# Vite and the React plugin
npm install --save-dev vite @vitejs/plugin-react

# SQLite library (choose one)
# For better performance and synchronous API:
npm install better-sqlite3

# Or, if you prefer the asynchronous API:
# npm install sqlite3

# For running multiple scripts concurrently
npm install --save-dev concurrently cross-env

# Electron DevTools installer
npm install --save-dev electron-devtools-installer
```

## **3. Set Up Project Structure**

Create the following directory structure:

```
electron-react-vite-sqlite-app/
├── src/
│   ├── main/
│   │   ├── main.js
│   │   └── preload.js
│   └── renderer/
│       ├── index.html
│       └── index.jsx
├── package.json
└── vite.config.js
```

## **4. Configure Vite**

Create `vite.config.js` in the root directory:

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  root: path.resolve(__dirname, 'src', 'renderer'),
  base: './',
  build: {
    outDir: path.resolve(__dirname, 'dist', 'renderer'),
    emptyOutDir: true,
  },
  server: {
    port: 3000,
    strictPort: true,
  },
})
```

## **5. Create the Electron Main Process**

Create `src/main/main.js`:

```javascript
// src/main/main.js
const { app, BrowserWindow, ipcMain } = require('electron')
const path = require('path')
const Database = require('better-sqlite3')
const isDev = process.env.NODE_ENV === 'development'
const { default: installExtension, REACT_DEVELOPER_TOOLS } = require('electron-devtools-installer')

let db

function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      contextIsolation: true,
    },
  })

  if (isDev) {
    win.loadURL('http://localhost:3000')
    win.webContents.openDevTools()

    // Install React DevTools
    installExtension(REACT_DEVELOPER_TOOLS)
      .then((name) => console.log(`Added Extension: ${name}`))
      .catch((err) => console.log('An error occurred: ', err))
  } else {
    win.loadFile(path.join(__dirname, '..', 'renderer', 'index.html'))
  }

  // Initialize SQLite database
  db = new Database('database.db')

  // Example IPC communication
  ipcMain.on('get-data', (event) => {
    const rows = db.prepare('SELECT * FROM your_table').all()
    event.reply('receive-data', rows)
  })
}

app.whenReady().then(createWindow)

app.on('window-all-closed', () => {
  if (db) db.close()
  if (process.platform !== 'darwin') app.quit()
})
```

## **6. Create the Preload Script**

Create `src/main/preload.js`:

```javascript
// src/main/preload.js
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld('api', {
  send: (channel, data) => {
    ipcRenderer.send(channel, data)
  },
  receive: (channel, func) => {
    ipcRenderer.on(channel, (event, ...args) => func(...args))
  },
})
```

## **7. Set Up the React Renderer**

Create `src/renderer/index.html`:

```html
<!-- src/renderer/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Electron React Vite App</title>
</head>
<body>
  <div id="root"></div>
  <script type="module" src="/index.jsx"></script>
</body>
</html>
```

Create `src/renderer/index.jsx`:

```jsx
// src/renderer/index.jsx
import React, { useEffect } from 'react'
import ReactDOM from 'react-dom'

function App() {
  useEffect(() => {
    window.api.send('get-data')
    window.api.receive('receive-data', (data) => {
      console.log('Data from SQLite:', data)
    })
  }, [])

  return <h1>Hello, Electron with React and Vite!</h1>
}

ReactDOM.render(<App />, document.getElementById('root'))
```

## **8. Update package.json Scripts**

Modify your `package.json`:

```json
{
  "name": "electron-react-vite-sqlite-app",
  "version": "1.0.0",
  "main": "src/main/main.js",
  "scripts": {
    "dev": "concurrently \"vite\" \"cross-env NODE_ENV=development electron .\"",
    "build": "vite build",
    "start": "electron ."
  },
  "devDependencies": {
    // ... your dev dependencies
  },
  "dependencies": {
    // ... your dependencies
  }
}
```

## **9. Handling SQLite in the Main Process**

In your `main.js`, ensure that all database operations are handled in the main process. Use IPC to communicate between the renderer and the main process for any data retrieval or manipulation.

Example of handling a database query in `main.js`:

```javascript
ipcMain.on('fetch-users', (event) => {
  const users = db.prepare('SELECT * FROM users').all()
  event.reply('fetch-users-response', users)
})
```

And in your React component:

```jsx
useEffect(() => {
  window.api.send('fetch-users')
  window.api.receive('fetch-users-response', (users) => {
    setUsers(users)
  })
}, [])
```

## **10. Secure IPC Communication**

Ensure that your IPC communication is secure by validating channels and data.

```javascript
// In preload.js
const validChannels = ['get-data', 'fetch-users']

contextBridge.exposeInMainWorld('api', {
  send: (channel, data) => {
    if (validChannels.includes(channel)) {
      ipcRenderer.send(channel, data)
    }
  },
  receive: (channel, func) => {
    if (validChannels.includes(channel)) {
      ipcRenderer.on(channel, (event, ...args) => func(...args))
    }
  },
})
```

## **11. Configure Electron Builder for Packaging**

If you plan to package your app, you can use `electron-builder`. Install it:

```bash
npm install --save-dev electron-builder
```

Add the build configuration to your `package.json`:

```json
{
  // ... other configurations
  "build": {
    "appId": "com.yourapp.id",
    "files": [
      "dist/**/*",
      "node_modules/**/*",
      "package.json",
      "src/main/**/*"
    ],
    "directories": {
      "buildResources": "assets",
      "output": "release"
    },
    "extraMetadata": {
      "main": "src/main/main.js"
    }
  }
}
```

Update the build script:

```json
"scripts": {
  // ... other scripts
  "package": "npm run build && electron-builder"
}
```

## **12. Run and Build Your App**

### **For Development:**

```bash
npm run dev
```

This script runs both the Vite dev server and Electron concurrently.

### **For Production Build:**

```bash
npm run build
```

This compiles your React app into static files in the `dist` directory.

### **To Package the App:**

```bash
npm run package
```

This packages your app into an executable.

---

**Note:** Ensure that you handle the environment variables correctly across different operating systems. The `cross-env` package helps set `NODE_ENV` in a cross-platform way.

## **13. Additional Tips**

- **Security:** Always be cautious with `contextIsolation`, `nodeIntegration`, and `enableRemoteModule` settings. It's recommended to keep `contextIsolation: true` and `nodeIntegration: false` for security.

- **Hot Reloading:** Vite's fast refresh should work out of the box with React, providing a smooth development experience.

- **Database Location:** Decide where to store your SQLite database file. For production apps, you might want to store it in a user data directory instead of the app directory.

- **Error Handling:** Add proper error handling for your database operations and IPC communications to prevent your app from crashing unexpectedly.

---

By following these steps, you should have a functional Electron app that uses SQLite for data storage, React for the frontend with React DevTools enabled, and Vite as the build tool. This setup provides a modern development experience with fast builds and reloads.

If you encounter any issues or need further clarification on any of the steps, feel free to ask!
