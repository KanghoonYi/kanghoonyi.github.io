---
title: Running Puppeteer on AWS Lambda
author: KanghoonYi
name: KanghoonYi(pour)
link: https://github.com/KanghoonYi/lambda-puppeteer
date: 2022-02-05 20:55:00 +0900
categories: [AWS, Lambda]
tags: [aws, lambda, Puppeteer]
pin: false
---

## What is Puppeteer?
> Puppeteer is a Node library which provides a high-level API to control Chrome or Chromium over the [DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/). Puppeteer runs [headless](https://developers.google.com/web/updates/2017/04/headless-chrome) by default, but can be configured to run full (non-headless) Chrome or Chromium.

## What is AWS Lambda?
> AWS Lambda is serverless computing service.

## Use Puppeteer on AWS Lambda
AWS Lambda has [limitation](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html#function-configuration-deployment-and-execution) about source code size.(Headless-Chrome installed with Puppeteer > 280MB )
So, Lambda run with Docker Image based on 'amazon/aws-lambda-nodejs:14'

### Repository
[https://github.com/KanghoonYi/lambda-puppeteer](https://github.com/KanghoonYi/lambda-puppeteer)

### Reference
- Puppeteer Repository
    - [Chrome headless doesn't launch on UNIX](https://github.com/puppeteer/puppeteer/blob/main/docs/troubleshooting.md#chrome-headless-doesnt-launch-on-unix)
- AWS Blog
    - [Field Notes: Scaling Browser Automation with Puppeteer on AWS Lambda with Container Image Support](https://aws.amazon.com/ko/blogs/architecture/field-notes-scaling-browser-automation-with-puppeteer-on-aws-lambda-with-container-image-support/)
- AWS Documentation
    - [Creating Lambda container images](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html)
