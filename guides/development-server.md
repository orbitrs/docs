# Orbit Development Server

This guide provides comprehensive documentation on the Orbit Development Server, a powerful tool for streamlining your development workflow when building Orbit Framework applications.

## Overview

The Orbit Development Server provides a complete development environment for Orbit applications, featuring:

- HTTP and WebSocket servers for serving your application
- Hot Module Replacement (HMR) for instant updates 
- File watching for automated rebuilds
- Advanced proxy capabilities for API integration
- Support for HTTPS and custom configurations

## Basic Usage

Start the development server with the `orbiton dev` command:

```bash
# Start the development server with default settings
orbiton dev

# Start on a custom port
orbiton dev --port 8080

# Start and automatically open in browser
orbiton dev --open
```

By default, the server runs on port 8000, but you can customize this with the `--port` option.

## Configuration Options

### Command Line Options

| Option | Description |
| ------ | ----------- |
| `--port`, `-p` | Specify the HTTP server port (default: 8000) |
| `--host`, `-h` | Specify the host to bind to (default: 0.0.0.0) |
| `--dir`, `-d` | Specify the project directory (default: current directory) |
| `--open`, `-o` | Automatically open in the default browser |
| `--https` | Enable HTTPS with auto-generated certificates |
| `--cert-file` | Path to custom SSL certificate (requires `--https`) |
| `--key-file` | Path to custom SSL key (requires `--https`) |
| `--hmr-verbose` | Enable verbose logging for Hot Module Replacement |
| `--no-hmr` | Disable Hot Module Replacement |
| `--proxy-target` | URL to proxy API requests to |
| `--proxy-path` | Path pattern to match for proxying (e.g., `/api`) |
| `--renderer` | Specify renderer to use (skia, wgpu, or auto) |

### Configuration File

You can also configure the development server using the `orbit.config.json` file in your project root:

```json
{
  "dev": {
    "port": 8080,
    "host": "localhost",
    "https": true,
    "certificates": {
      "cert": "./certs/cert.pem",
      "key": "./certs/key.pem"
    },
    "hmr": {
      "enabled": true,
      "verbose": false
    },
    "proxy": [
      {
        "path": "/api",
        "target": "http://localhost:3000",
        "changeOrigin": true,
        "secure": false
      }
    ],
    "open": true,
    "renderer": "auto"
  }
}
```

## Architecture

The development server consists of two main components:

1. **HTTP Server**: Serves static files and handles API requests
2. **WebSocket Server**: Enables real-time communication for HMR and live reloading

The WebSocket server automatically runs on one port higher than the HTTP server (e.g., if HTTP is on port 8000, WebSocket runs on port 8001).

## Hot Module Replacement (HMR)

Hot Module Replacement allows your application to update without a full page refresh when you make changes to your code.

### How It Works

1. The development server watches for file changes
2. When changes are detected, the server compiles only the affected modules
3. Updated modules are sent to the client via WebSocket
4. The client replaces the old modules with the new ones and preserves application state

### Client Integration

The HMR client is automatically injected into your application during development. You can also add custom HMR handling:

```rust
#[cfg(debug_assertions)]
pub fn register_hmr_handlers() {
    orbit::hmr::on_update(|module_id| {
        if module_id == "components/counter" {
            // Custom handling for Counter component updates
            println!("Counter component updated!");
        }
    });
}
```

## Proxy Configuration

The development server can proxy API requests to another server, which is useful for local development with separate frontend and backend servers.

### Basic Proxy Configuration

```json
{
  "dev": {
    "proxy": [
      {
        "path": "/api",
        "target": "http://localhost:3000"
      }
    ]
  }
}
```

### Advanced Proxy Options

```json
{
  "dev": {
    "proxy": [
      {
        "path": "/api",
        "target": "http://localhost:3000",
        "changeOrigin": true,
        "secure": false,
        "ws": true,
        "pathRewrite": {
          "^/api": ""
        },
        "headers": {
          "X-Custom-Header": "Orbit-Dev-Server"
        }
      }
    ]
  }
}
```

## HTTPS Support

For secure development, especially when working with features that require HTTPS:

```bash
# Enable HTTPS with auto-generated certificates
orbiton dev --https

# Use custom certificates
orbiton dev --https --cert-file ./certs/cert.pem --key-file ./certs/key.pem
```

### Trusting Auto-Generated Certificates

When using auto-generated certificates, you may need to add them to your system's trusted certificates:

1. Auto-generated certificates are stored in `.orbit/certs/`
2. Add the certificate to your system's trusted certificates store
3. For development purposes only - never use auto-generated certificates in production

## Static File Serving

The development server serves static files from your project directory:

- Root files (like `index.html`) are served from the project root
- Static assets are served from the `/public` directory
- Built files are served from the `/dist` directory

## WebSocket API

The WebSocket server can be used for custom development-time features:

```javascript
// Connect to the dev server WebSocket
const wsPort = window.location.port ? parseInt(window.location.port) + 1 : 8001;
const wsUrl = `${window.location.protocol === 'https:' ? 'wss:' : 'ws:'}//${window.location.hostname}:${wsPort}`;
const socket = new WebSocket(wsUrl);

// Handle messages
socket.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  switch (data.type) {
    case 'fileChange':
      console.log('Files changed:', data.paths);
      break;
    case 'rebuild':
      console.log('Rebuild status:', data.status);
      break;
    default:
      console.log('Unknown message:', data);
  }
};
```

## Custom Middleware

Advanced users can add custom middleware to the development server by creating a `dev-server.js` file in the project root:

```javascript
// dev-server.js
module.exports = {
  // Before built-in middleware
  before(app, server) {
    app.get('/custom', (req, res) => {
      res.send('Custom middleware response');
    });
  },
  
  // After built-in middleware
  after(app, server) {
    app.use((req, res, next) => {
      console.log(`Request: ${req.method} ${req.url}`);
      next();
    });
  }
};
```

## Environment Variables

The development server loads environment variables from `.env` files:

- `.env` - Default environment variables
- `.env.development` - Development-specific variables
- `.env.local` - Local overrides (not committed to version control)

Environment variables are accessible in your application code through the `orbit::env` module:

```rust
let api_url = orbit::env::get("API_URL").unwrap_or_else(|| "http://localhost:3000".to_string());
```

## Performance Monitoring

Monitor the performance of your development server:

```bash
# Enable performance monitoring
orbiton dev --perf-monitor
```

Access the performance dashboard at `http://localhost:[PORT]/__performance` to see:

- Memory usage
- CPU usage
- Network latency
- Build times
- Module sizes

## Troubleshooting

### Common Issues

1. **Port already in use**
   ```
   Error: Failed to bind to port 8000
   ```
   Solution: Use a different port with `--port` option

2. **WebSocket connection failed**
   ```
   WebSocket connection to 'ws://localhost:8001/' failed
   ```
   Solution: Check firewall settings or network configuration

3. **HMR not working**
   ```
   Warning: HMR update failed for module 'components/counter'
   ```
   Solution: Try disabling and re-enabling HMR, or restart the server

4. **HTTPS certificate issues**
   ```
   Error: self-signed certificate
   ```
   Solution: Add the certificate to your system's trusted certificates store

### Logs and Debugging

To enable verbose logging:

```bash
orbiton dev --hmr-verbose
```

Logs are also written to `.orbit/logs/dev-server.log` for later inspection.

## Advanced Use Cases

### Multiple Servers

Run multiple development servers for different projects:

```bash
# Terminal 1 - Main app
orbiton dev --port 8000 --dir ./main-app

# Terminal 2 - Component library
orbiton dev --port 8080 --dir ./component-lib
```

### Custom Build Pipeline

Integrate with custom build tools:

```bash
# Run custom build steps before starting dev server
npm run build:custom && orbiton dev
```

### CI/CD Integration

For automated testing environments:

```bash
# Start in CI-friendly mode (no interactive output)
orbiton dev --ci --port 8080 --no-hmr
```

## API Reference

For programmatic usage of the development server, see the [Orbiton Library API documentation](./orbiton-lib-api.md).
