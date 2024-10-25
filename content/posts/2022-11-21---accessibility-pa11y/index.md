---
title: "Automate your e2e accessibility tests using pa11y"
template: "post"
category: "Accessibility"
tags:
  - "Nodejs"
  - "Pa11y"
  - "Puppeteer"
  - "A11y"
published: true
draft: false
date: "2022-11-21T23:46:37.121Z"
slug: "/posts/accessbility-pa11y"
description: "I'm excited to announce the new [pa11y-e2e-tests](https://www.npmjs.com/package/pa11y-e2e-tests) framework, which helps you find accessibility issues of any web app. This enhancement is simple, fast, reliable, and easy to integrate with your existing e2e tests. "
---

I'm excited to announce the new [pa11y-e2e-tests](https://www.npmjs.com/package/pa11y-e2e-tests) framework, which helps you find accessibility issues of any web app. This enhancement is simple, fast, reliable, and easy to integrate with your existing e2e tests. 

But before we jump into this interesting framework, lets summarise accessibility testing in simple words.

## What is accessibility testing?

Accessibility testing ensures that differently-abled people can access the information displayed on the application as people without disabilities.

## Why it is important?

Improving the accessibility of your product can attract the usability of any application, including those with low vision, blindness, hearing impairment, cognitive impairment, motor impairments, or situational disabilities. 

## Process of finding a11y issues in any application?

There are a set of rules/guidelines which are defined by [WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/) (Web Content Accessibility Guidelines), which needs to be followed while developing any web application. Mention rules within the guidelines help design a website that can increase customer engagement despite disabilities. 

Manually understanding and finding accessibility issues within an application can be quite tedious. Hence there are various tools in the market which help resolve this problem. 

This framework is built using [Pa11y](https://pa11y.org/) which is a free and open-source tool helping out the development team to build and design accessible web pages.

## How to use [pa11y-e2e-tests](https://www.npmjs.com/package/pa11y-e2e-tests)?

This package identifies accessibility issues, which can be integrated with your existing e2e tests.

### Installation

Install the package, using the command: 

`npm install pa11y-e2e-tests --save`

### Configure

Before getting started with accessibility testing, we need to be mindful of a few things like 

1. `browser` - for launching the application in an automated way, so that pa11y identifies the a11y issues
2. `launch_url`: contains the default URL to be launched by the browser
3. `src_folders`: Need to define the folder name which contains tests to be run
4. `reports_path`: Location to be specified for test results to be saved
5. `reporter`: The type of report required, currently supports `JSON` and `HTML`

And many more, Pa11y has lots of options you can use to change the way the browser runs or the way page is loaded (reference [link](https://github.com/pa11y/pa11y#configuration) ). Below listed are the default settings that have been passed during the test execution.

| Name | Types | Default | description |
| --- | --- | --- | --- |
| runners | array | none | An array of runner names that correspond to existing and installed pa11y runners, If a runner is not found then pa11y will throw an error |
| standard | string | WCAG2AA | The accessibility standard to use when testing pages. This should be one of WCAG2A, WCAG2AA, or WCAG2AAA. Note: only used by htmlcs runner |
| timeout | integer | none | The time in milliseconds that a test should be allowed to run before calling back with a timeout error. Please note that this is the timeout for the entire test run (including time to initialize Chrome, load the page, and run the tests) |
| includeNotices | boolean | false | Whether to include results with a type of notice in the Pa11y report. Issues with a type of notice are not directly actionable and so they are excluded by default. You can include them by using this option |
| includeWarnings | boolean | false | Whether to include results with a type of warning in the Pa11y report. Issues with a type of warning are not directly actionable and so they are excluded by default. You can include them by using this option |

Let’s configure our framework by creating a file with the name `pa11y.config.js`  and add respective information to get started with a11y testing.

```js
module.exports = {
  launch_url: "https://www.saucedemo.com",
  src_folders: "tests",
  reports_path: "reports",
  reporter: ["json", "html"],
  test_settings: {
    runners: [
      'axe',
      'htmlcs'
    ],
    standard: 'WCAG2A',
    timeout: 120000,
    includeNotices: true,
    includeWarnings: true
  },
  puppeteer_settings: {
    headless: false,
    executablePath: '/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome'
  }
};
```

Note: `puppeteer_settings` is passed internally to launch the browser and perform necessary actions on the webpage.

### Let’s get started

1. Create a first test file under the `tests` folder, and name the file as `test.js`

```js
module.exports = {
  url: `/`,
  actions: [
    'set field #username to standard_user',
    'set field #password to secret_sauce',
    'click element input[name=login-button]',
    'wait for element div[class="app_logo"] to be visible'
  ]
}
```

The above code launches a browser on the URL defined within the config file. It then performs the actions required to log in to the site. Like entering a username, and password on the login page and clicking on the login button. 

These tests will generate a report summarising the total number of accessibility issues existing on the homepage screen (after the login screen).

1. Add the command to run the accessibility tests inside your `package.json`

```js
"scripts": {
	"test": "runAccessibility"
}
```

The below command will run all the tests placed under `tests` folder

```js
npm run test
```

You can also run a single file, using the command 

```js
npm run test --test tests/firstTest.js
```

1. Here you go, reports folder will contain all the test results under it. It would have both `JSON` and `HTML` reports

I hope you find this package useful. Happy reading 🙂

Resources:
- [Pa11y website](https://pa11y.org/)
- [WCAG guidelines](https://www.w3.org/WAI/standards-guidelines/wcag/)
- [Why websites should be accessible](https://www.iweb.co.uk/2016/10/inclusive-design-why-our-websites-should-more-accessible/)
- [Another useful blog explaining accessibility testing manually using different tools](https://medium.com/pulsar/which-accessibility-testing-tool-should-you-use-e5990e6ef0a)
