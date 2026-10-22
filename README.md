<p align="center">
  <a href="https://solvecha.com"><img src="https://solvecha.com/site/static/apple-touch-icon.png" width="64" height="64" alt="Solvecha"></a>
</p>

<h1 align="center">Solvecha browser extension</h1>

<p align="center">
  Solves reCAPTCHA v2 image challenges in Chrome, Chromium, Playwright, Puppeteer and Selenium.<br>
  Uses your <a href="https://solvecha.com">Solvecha</a> API key. Free tier: 100 solves per day.
</p>

---

## Download

Get the latest `solvecha-chrome-<version>.zip` from [Releases](https://github.com/Solvecha/solvecha-extension/releases/latest) and unzip it.

```bash
curl -sL -o solvecha.zip https://github.com/Solvecha/solvecha-extension/releases/latest/download/solvecha-chrome.zip
unzip -q solvecha.zip -d solvecha-extension
```

Each release lists the SHA-256 of the zip.

## Get an API key

1. Create an account on [solvecha.com](https://solvecha.com/register). No card needed.
2. Copy your key from the [dashboard](https://solvecha.com/dashboard).

## Chrome or Chromium by hand

1. Open `chrome://extensions` and turn on **Developer mode**.
2. Click **Load unpacked** and pick the unzipped folder.
3. Open the extension's settings and paste your key.

## Playwright

Extensions need a headed browser or a virtual display such as `xvfb-run`. Use Chromium, Chrome for Testing or the Chromium that Playwright installs. Google Chrome 137 and newer ignores `--load-extension`.

```js
import { chromium } from "playwright";

const ext = "/path/to/solvecha-extension"; // the unzipped folder
const context = await chromium.launchPersistentContext("/tmp/profile", {
  headless: false,
  args: [`--disable-extensions-except=${ext}`, `--load-extension=${ext}`],
});

// Save the key once. It stays in the profile folder.
let [worker] = context.serviceWorkers();
worker ??= await context.waitForEvent("serviceworker");
await worker.evaluate((key) => chrome.storage.local.set({ apiKey: key }), "YOUR_API_KEY");

const page = await context.newPage();
await page.goto("https://www.google.com/recaptcha/api2/demo");
// The extension solves the challenge. Wait for the result you need.
```

## Puppeteer

```js
import puppeteer from "puppeteer";

const ext = "/path/to/solvecha-extension"; // the unzipped folder
const browser = await puppeteer.launch({
  headless: false,
  args: [`--disable-extensions-except=${ext}`, `--load-extension=${ext}`],
});

// Save the key once on the settings page.
const worker = await browser.waitForTarget((t) => t.type() === "service_worker");
const id = new URL(worker.url()).host;
const settings = await browser.newPage();
await settings.goto(`chrome-extension://${id}/options.html`);
await settings.type("#key", "YOUR_API_KEY");
await settings.click("#save-key");

const page = await browser.newPage();
await page.goto("https://www.google.com/recaptcha/api2/demo");
```

## Selenium

```python
from selenium import webdriver

ext = "/path/to/solvecha-extension"  # the unzipped folder
options = webdriver.ChromeOptions()
options.add_argument(f"--load-extension={ext}")
options.add_argument("--user-data-dir=/tmp/profile")  # keeps the saved key

driver = webdriver.Chrome(options=options)
# First run: open the extension's settings page once and save your key.
driver.get("https://www.google.com/recaptcha/api2/demo")
```

Wait for the token before you submit a form: the field `g-recaptcha-response` holds a long token when the challenge is solved.

## Tips

- **Fewer challenges:** run the extension in [CloakBrowser](https://solvecha.com/docs/cloakbrowser), a Chromium build with a real-looking fingerprint.
- **No browser?** Use the [API](https://solvecha.com/docs) directly, in sync mode for one-request answers, or the [MCP server](https://solvecha.com/docs/ai-agents) for AI agents.
- **Settings:** delays, auto-open, excluded sites and the Google-feedback switch are on the extension's settings page.

## What the extension sends

Only to `api.solvecha.com`: the challenge pictures and the task text, and after each round whether Google accepted it (you can turn this off). No page addresses, no cookies. See the [privacy policy](https://solvecha.com/privacy).

## Links

[Website](https://solvecha.com) · [Docs](https://solvecha.com/docs/browser-extension) · [Pricing](https://solvecha.com/pricing) · [Support](mailto:hello@marin.sh)

<sub>This repository contains release builds only. © 2026 Marin Solutions UG (haftungsbeschränkt). Free to use with a Solvecha account; redistribution of modified builds is not allowed. Not affiliated with Google.</sub>
