# Using Playwright with Neko Chromium

## Quick Start

```bash
docker-compose up
```

Exposes:
- Port 8080: Neko web interface
- Port 9222: Chrome DevTools Protocol
- Port 9223: CDP proxy (if using socat)

## Playwright Connection

```javascript
import { chromium } from 'playwright';

const browser = await chromium.connectOverCDP('http://localhost:9222');

// Create new context with video recording
const context = await browser.newContext({
  recordVideo: {
    dir: './videos',
    size: { width: 1920, height: 1080 }
  },
  viewport: { width: 1920, height: 1080 }
});

// Create new page with recording
const page = await context.newPage();

// Your test/automation code here
await page.goto('https://github.com');

// Close context to save video
await context.close();
```
