# Puppeteer cluster Connect

Ase Cluster.connect instead on Cluster.launch

## Installation

Install using your favorite package manager:

```sh
npm install --save puppeteer # in case you don't already have it installed
npm install --save puppeteer-cluster-connect
```

Alternatively, use `yarn`:

```sh
yarn add puppeteer puppeteer-cluster-connect
```

Current support : one restart function

Array restart function is current in development

## Usage

The following is a typical example of using puppeteer-cluster. A cluster is created with 2 concurrent workers. Then a task is defined which includes going to the URL and taking a screenshot. We then queue two jobs and wait for the cluster to finish.

```js
const { Cluster } = require("../dist");
const puppeteer = require("puppeteer-core"); // use puppeteer-core instead of puppeteer
const puppeteerRemote = require("puppeteer");
// delay function
const ws = async () => {
    console.log("create browser");
    const browser = await puppeteerRemote.launch({
        headless: false,
        defaultViewport: null,
        args: [
            "--start-maximized", // you can also use '--start-fullscreen'
            "--disable-web-security",
            "--disable-site-isolation-trials",
            "--disable-application-cache",
            // "--user-data-dir=C:\\Users\\vukhaihoan\\AppData\\Local\\Google\\Chrome\\User Data",
            // "--profile-directory=Profile 2",
        ],
        executablePath:
            "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe",
    });

    // Store the endpoint to be able to reconnect to Chromium
    const browserWSEndpoint = browser.wsEndpoint();
    console.log(browserWSEndpoint);
    // Disconnect puppeteer from Chromium
    browser.disconnect();
    return browserWSEndpoint;
};
(async () => {
    try {
        const cluster = await Cluster.connect({
            concurrency: Cluster.CONCURRENCY_BROWSER,
            maxConcurrency: 2,

            // provide the puppeteer-core library
            puppeteer,
            // and provide executable path (in this case for a Chrome installation in Ubuntu)
            puppeteerOptions: {
                // executablePath: 'google-chrome-stable',
                // headless: false,
                browserWSEndpoint: await ws(),
                // defaultViewport: null,
            },
            restartFunction: ws,
            // perBrowserOptions: [
            //     {
            //         browserWSEndpoint: await ws(),
            //     },
            //     {
            //         browserWSEndpoint: await ws(),
            //     },
            //     {
            //         browserWSEndpoint: await ws(),
            //     },
            // ],
        });

        await cluster.task(async ({ page, data: url }) => {
            console.log("chay : " + url);
            await page.goto(url);
            console.log("went to: " + url);
        });

        cluster.queue("https://www.google.com");
        cluster.queue("https://www.wikipedia.org");
        cluster.queue("https://github.com/");
        await cluster.idle();
        await cluster.close();
    } catch (error) {
        console.log(error);
    }
})();
```
