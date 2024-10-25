---
title: "Run front-end performance test using lighthouse with puppeteer + Git Actions"
template: "post"
category: "Lighthouse"
tags:
  - "git"
  - "puppeteer"
  - "nodejs"
  - "javascript"
  - "lighthouse"
  - "gatsby"
published: true
draft: false
date: "2021-05-11T23:46:37.121Z"
slug: "/posts/lighthouse-with-puppeteer"
description: "Boost your website’s performance with an automated testing setup using Lighthouse, Puppeteer, and GitHub Actions! In this blog, we’ll guide you through setting up a pipeline that runs front-end performance tests automatically on every push. By integrating Lighthouse with Puppeteer, you’ll simulate real user interactions, capturing accurate performance metrics and insights. Then, automate the process using GitHub Actions to ensure every code change is tested for speed, accessibility, and best practices. Follow along to streamline your workflow and keep your front-end optimized without lifting a finger! "
---

## What is front-end performance?

In simple words, it's nothing but the experience of the product for end users. Mainly describes the performance of your product.

### Why front-end performance is important?

Earlier, the front-end performance was not necessarily important as the pages use to simply load the html. However the need urged when we adopted use of CSS, Ajax, dynamic images as the web page became a collection of such resources. So if we ignore the front-end performance testing, the sufferings would be to the product adheres to the business.

Billions of websites are browsed every minute, and the half of the users just closes them within seconds. Just because the users are not likely to spend their time on slow loading sites.

The ultimate desire of front-end performance is to provide what they need from the site, within no time.

### Factors impacting the performance

There are various factors impacting the performance of any site. I tried listing some of them below:
- Unused CSS
- Unoptimized images
- Minimizing the http requests
- Cookie size
- Non-blocking javascripts
- Javascripts profiling
- And lots more

### Different tools are available for testing the front-end performance

Below are some listed:
- Lighthouse
- Web Page Test
- Pingdom
- Jmeter

My personal favorite for front-end testing is Lighthouse. As it just doesn't tell you about the problem but also provides possible solutions to fix these problems.

### What is puppeter?

Puppeteer is NodeJS library that provides a high-level API control over special devTools protocol. By default its runs in headless mode and had multiple advantages for test automation for web applications & scraping websites for data.

### Configure Lighthouse

Lighthouse is integrated to Google Chrome. After collecting performance-related insights, the tool automatically compiles a report into a HTML support. Apart from presenting all collected data in a clear, concise way each report has a section dedicated to potential fixes of the detected issues.

- Install Lighthouse

```
yarn add lighthouse
```

- Install Puppeteer

```
yarn add puppeteer
```

- Folder structure

```
  perf
  ├─ reports
      ├─ loginPageforWeb.html
      ├─ homePageforweb.html
  └─ config
      ├─ siteConfig.json
      ├─ lighthouseSettings.json
  └─ utils
      ├─ auth.js
  └─ generatePerfReport.js

```

- Lighthouse provides various options to configure as per convinces of your site. Which could be passed into a lighthouseSettings.json file something like I did below:

```
 {
  "extends": "lighthouse:default",
  "settings": {
    "onlyCategories": [
      "accessibility",
      "best-practices",
      "performance",
      "seo"
    ],
    "formFactor": "desktop",
    "screenEmulation": {
      "mobile": false,
      "width": 1350,
      "height": 940,
      "deviceScaleFactor": 1,
      "disabled": false
    },
    "emulatedUserAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4420.0 Safari/537.36 Chrome-Lighthouse"
  }
}
```

- Keep all the site related variables into one file, like siteConfig.json:

```
{
  "dev": {
    "originUrl": "https://abc.com",
    "targetUrl: "https://abc.com/home",
    "username": "efg",
    "password": "Password123"
  }
}
```

### Launch browser using puppeteer and run our first test

Here is the file `generatePerfReport.js` which generates performance report of any random site.

```
const puppeteer = require('puppeteer');
const lighthouse = require('lighthouse');

const settings = require('./config/lighthouseSettings.json')

const PORT = 8041;

const options = {
  port: PORT,
  disableStorageReset: true,
  output: 'html',
};

const generatePerformanceReport = async () => {
  const browser = await puppeteer.launch({
    args: [`--remote-debugging-port=${PORT}`],
    headless: true, //can be set to false
    slowMo: 50,
  });

  const url = "https://abc.com"
  const loginPageReport = await lighthouse(url, options, settings);
  const loginPageReportHtml = loginPageReport.report;
  fs.writeFileSync(
    `./reports/loginPageReportfor.html`,
    loginPageReportHtml,
  );
  console.log('Performance report generated successfully!!');

  await browser.close();
};

if (require.main === module) {
  try {
    generatePerformanceReport();
  } catch (e) {
    console.log(e);
  }
} else {
  module.exports = generatePerformanceReport;
}

```

### Get the performance of authenticated sites using puppeteer

Let's create a test file `auth.js` which helps us to login to any authenticated site. It takes 2 parameter `browser` and `origin` which we will discuss on further

```
const config = require('config/siteConfig.json');

// config required inorder to login to the site
const EMAIL = config.username;
const PASSWORD = config.password;

// Test Steps to login the site with username & password

const login = async (browser, origin) => {
  const page = await browser.newPage();
  await page.goto(origin);
  await page.waitForSelector('input[id="username"]', {visible: true});
  await page.waitForSelector('input[id="username"]', {visible: true});
  const emailInput = await page.$('input[id="username"]');
  await emailInput.type(EMAIL);
  const nextBttn = await page.$('span[id="submit-text"]');
  await nextBttn.click();
  await page.waitForSelector('input[id="password"]', {visible: true});
  const passwordInput = await page.\$('input[id="password"]');
  await passwordInput.type(PASSWORD);
  await Promise.all([
    page.click('button[id="submit-login-form"]'),
    page.waitForNavigation(),
  ]);
  await page.close();
};

module.exports = login;

```

Now, let's enhance `generatePerfReport.js` to get performance of authenticated pages

```
const fs = require('fs');
const puppeteer = require('puppeteer');
const lighthouse = require('lighthouse');

const config = require('config/siteConfig.json');
const settings = require('./config/lighthouseSettings.json')
const login = require('./utils/auth');

const PORT = 8041;

const options = {
  port: PORT,
  disableStorageReset: true,
  output: 'html',
};

// Launching browser in headless mode to generate report for login & homePage
const generatePerformanceReport = async () => {
  const browser = await puppeteer.launch({
    args: [`--remote-debugging-port=${PORT}`],
    headless: true, //can be set to false
    slowMo: 50,
  });

  // Generating report for login page, which is an unauthenticated page
  await generateReportForLogin();

  // Logging into the site
  await login(browser, config.originUrl);

  // Generating report for home page, which is an authenticated page
  await generateReportForHome();

  await browser.close();
};

// Storing the login page report in file system
const generateReportForLogin = async () => {
  const loginPageReport = await lighthouse(config.originUrl, options, settings);
  const loginPageReportHtml = loginPageReport.report;
  fs.writeFileSync(
    `./reports/loginPageReport.html`,
    loginPageReportHtml,
  );
  console.log(`login page performance report generated successfully`);
};

// Storing the home page report in file system
const generateReportForHome = async () => {
  //HomePage performance
  const url = config.targetUrl;
  const homePageReport = await lighthouse(url, options, settings);
  const homePageReportHtml = homePageReport.report;
  fs.writeFileSync(
    `./reports/homePageReport.html`,
    homePageReportHtml,
  );
  console.log(`home page performance report generated successfully`);
};

if (require.main === module) {
  try {
    generatePerformanceReport();
  } catch (e) {
    console.log(e);
  }
} else {
  module.exports = generatePerformanceReport;
}
```

- Run your performance test, by using simple command:

```
node generatePerformanceReport.js
```

### How do we fix the performance related issues ?

There is a wonderful blog, which explains how such issues could fixed, continue reading for [reference](https://medium.com/@april9288/how-i-got-a-100-lighthouse-score-with-my-react-app-2b676ef62113).

### Where you run the front-end performance test?

Never run the performance test on local machine, as the results would always fluctuating. Due to various factors affecting the run.

- Internet traffic & routing changes
- Testing on different devices, such as a high-performance desktop and a low-performance laptop
- Browser extensions that inject JavaScript and add/modify network requests etc

Hence always run it in a stable enviroment. So lets do something exciting, lets generate our performance reports using Github Actions.

- Structure your github actions folders:

```

.github
├─ workflows
├─ runPerfActions.yaml

```
- Lets proceed with configuring our yaml file

```

name: Trigger front end perf
on:
  workflow_dispatch:
jobs:
  generatePerfReport:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: e2e/perf
    strategy:
      matrix:
        node-version: [12.x]
    steps:
    - uses: actions/checkout@v2
    - name: Install node ${{ matrix.node-version }}
      uses: actions/setup-node@v2
      with:
        node-version: ${{ matrix.node-version }}
    - uses: borales/actions-yarn@v2.0.0
    - run: yarn install
    - run: node generatePerfReport.js
```

_*Recommendation: You could also try posting your reports on Google drive. Shortly would be posting a article on how to upload reports to google drive*_

Thanks for reading 😄.
