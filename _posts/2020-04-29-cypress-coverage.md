---
title: "One coverage to rule them all"
description: "Generating combined coverage reports for Cypress end to end tests and Jest unit tests"
author: Thomas Zimmermann
tags: testing, code coverage, nyc, Jest, istanbul, Cypress, JavaScript
layout: article
image:
  feature: /2020/cypress.jpg
  alt: "Snowy mountain landscape with cypress trees and two snowboarders riding in a chairlift"
---

This post shows how different code coverage reports for [Cypress](https://www.cypress.io) end to end and [Jest](https://jestjs.io/) unit tests can be combined into one unified report.

This was motivated by a [talk](https://www.youtube.com/watch?v=JL3QKQO80fs) I've seen at [ReactiveConf 2019](https://reactiveconf.com/) in Prague from Gleb Bahmutov, one of the core maintainers of Cypress.
Among other interesting tidbits about testing with Cypress, he introduced a [code coverage plugin for Cypress](https://www.npmjs.com/package/@cypress/code-coverage) that could be used to generate coverage reports for end to end tests.

But first, let's have a look at why it's a good idea to collect code coverage in general.

## Why we collect coverage

During development, we strive to keep the Jest unit test coverage at the current level of ~80%.
But looking at the coverage number alone is not really helpful (and should not be a goal on its own). More importantly, we are looking for:

* __Sudden changes in coverage__: If a pull request decreases coverage signficantly, chances are that we either forgot to write tests, or we deleted/disabled existing tests by accident. It might also be that we forgot to adapt tests after some refactoring.
* __Uncovered parts__: Are there certain edge cases that are not covered but should be?

[Codecov](https://codecov.io/) greatly helps us with that by providing diff views on every pull request that highlight uncovered lines.

All in all, this gives us greater confidence in our *unit* tests, but what about our *end to end* tests with Cypress?

## How we combine unit and end to end test coverage

Following this [excellent tutorial](https://docs.cypress.io/guides/tooling/code-coverage.html#E2E-and-unit-code-coverage), it was straight forward to add the `@cypress/code-coverage` plugin to our Cypress setup.
Summarizing the main steps that we had to do:

### Add [babel-plugin-istanbul](https://github.com/istanbuljs/babel-plugin-istanbul) 

We installed the plugin and added this piece to our `.babelrc` file:

```json
    "env": {
        "development": {
            "plugins": [
                "istanbul"
            ]
        }
    }
```

This instruments our client JavaScript code, i.e. it adds a global `window.__coverage__` object.
We made sure to only add it in `development` mode, as it would add quite some overhead to the production code otherwise.

### Install `@cypress/code-coverage` plugin

To actually generate a coverage report for our Cypress tests, we need to install the plugin and its peer dependencies. *Note*: These instructions assume `v1.10.4` of the plugin. With the most recent release, all necessary deps are installed automatically.
```
yarn add -D  @cypress/code-coverage nyc istanbul-lib-coverage
```
Then we register the plugin with Cypress as shown [here](https://github.com/cypress-io/code-coverage/blob/master/README.md#install).
Now, whenever we run the Cypress end to end tests, we will have a coverage report generated.

### Add CI pipeline steps

We would like to have both coverage reports merged and reported to any open pull request.
So in our CI pipeline (using CircleCI), we have a step `collect-test-coverage` that runs after both, the end to end tests and the unit tests have finished.
It performs the following steps:

1. Copy both coverage reports into a single folder
2. Run `nyc merge` on that folder
3. Upload the resulting `coverage.json` using [codecov](https://github.com/codecov/codecov-node)

The steps are basically a simplified version of [this example repo](https://github.com/bahmutov/cypress-and-jest).

## Results

We observed a ~10% increase in code coverage when looking at the combined code coverage (right hand chart), compared to only unit test coverage (left hand chart).

![Visualised code coverage after merging end to end ad unit test coverage reports](../2020/coverage-before.svg)
![Visualised code coverage after merging end to end ad unit test coverage reports](../2020/coverage-after.svg)

### What does this mean for us?

In general, end to end tests cover a lot more lines of code than unit tests, because they are spinning up the whole application and testing lots of different interacting components usually.
This explains why we gained an additional 10% on top of our already high code coverage.
But this does not mean that we're not writing unit tests any more.
Writing unit tests is still mandatory because of the following reasons:

* They are faster to write.
* They run faster (locally and on CI).
* They are faster to debug.
* There are certain error cases that cannot be tested via end to end tests, because they are not easily reachable from the UI.

So our usual flow still consists of writing lots of unit tests, making sure coverage does not drop significantly, and adding end to end tests afterwards to test user flows.

But in the end, it's still  nice to have one central overview of how much of our code is covered by any kind of test.


<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@yann_allegre?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Yann Allegre"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Yann Allegre</span></a>
