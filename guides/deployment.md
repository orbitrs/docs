# 🚀 Deployment Guide

## 📝 Overview

This guide covers how to prepare, build, and deploy your Orbit applications to various environments, including static hosting, server environments, desktop packages, and embedded targets.

## 🌐 Static Hosting (Web)

1. Build for production:

```bash
orbiton build --target web
```

2. Output directory: `dist/`
3. Upload `dist/` contents to any static host (Netlify, Vercel, GitHub Pages).

## 🖥 Desktop Packaging

1. Install packaging tool:

```bash
npm install -g electron-packager
```

2. Configure `package.json` and `orbiton.config.js`.
3. Package application:

```bash
electron-packager ./dist MyApp --platform=darwin,linux,win32 --arch=x64
```

## 📱 Mobile Deployment

1. Enable mobile adapter in `orbiton.config.js`:

```toml
[build]
target = ["web", "ios", "android"]
```

2. Build for mobile:

```bash
orbiton build --target ios
orbiton build --target android
```

3. Use Xcode or Android Studio to open and deploy.

## 🛠 Server-Side Hosting

1. Build server bundle:

```bash
orbiton build --target server
```

2. Deploy with Node.js:

```bash
node dist/server/index.js
```

## 📦 Embedded Systems

1. Enable embedded targets in `orbiton.config.js`.
2. Build with: `orbiton build --target embedded`
3. Flash to device using vendor-specific tools.

## 🔄 Continuous Deployment

Set up CI/CD pipelines to automate builds and deployments:

```yaml
# Example GitHub Actions
name: Deploy Web
on:
  push:
    branches: [ main ]
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cargo install orbiton
      - run: orbiton build --target web
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

## 📄 Best Practices

- Use environment variables for secrets
- Minify assets and enable gzip/Brotli
- Monitor performance and errors post-deployment

## 🔗 Related Guides

- [Performance Optimization](./performance-optimization.md)
- [Continuous Integration](./testing.md)
