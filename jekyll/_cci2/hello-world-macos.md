---
layout: classic-docs
title: "Hello World on macOS"
short-title: "Hello World"
description: "First project on CircleCI 2.0"
categories: [getting-started]
order: 4
---

This document describes how to get started with continuous integration on **macOS
build environments** on CircleCI. If you still need to get acquainted with
CircleCI, it is recommended to checkout the [getting started guide]({{ site.baseurl }}/2.0/getting-started).

## Prerequisites

To follow along with this document you will need:

- An [account](https://circleci.com/signup/) on CircleCI.
- A subscription to a [paid plan](https://circleci.com/pricing/#build-os-x) to enable building on the macOS executor.
- An Apple computer with XCode installed on it (if you want to open the example project).

## Overview to the macOS executor

The macOS build environment (or 'executor') is used for iOS and macOS
development, allowing you to test, build, and deploy macOS and iOS applications on
CircleCI. The macOS executor runs jobs in a macOS environment on a VM and provides access to iOS simulators.

Before we get to setting up the macOS executor, we will need to setup our example application.

## Example Application

The example application is a simple mac app - it runs a 5 minute
timer and contains a single unit test (real-world applications
will be far more complex; this app simply serves as an introduction to the macOS
build environment).

As a user getting to know the macOS build environment, our ideal scenario is for CircleCI to help with the following:

- Run tests using XCode on the macOS VM whenever we push code.
- Create and upload the compiled application as an artifact after tests have run successfully.

You can checkout the example application's repo on
[Github](https://github.com/CircleCI-Public/circleci-demo-macos).

## Example Configuration File

Our application does not make use of any external tools or dependencies, so we
have a fairly simple `.circleci/config.yml` file. Below, each line is commented
to indicate what is happening at each step.

```yaml
version: 2 # use version 2.0 of CircleCI
jobs: # a basic unit of work in a run
  build: # runs not using `Workflows` must have a `build` job as entry point
    macos:  # indicate that we are using the macOS executor
      xcode: "10.0.0" # indicate our selected version of Xcode
    steps: # a series of commands to run
      - checkout  # pull down code from your version control system.
      - run:
          # run our tests using xcode's cli tool `xcodebuild`
          name: Run Unit Tests
          command: xcodebuild test -scheme circleci-demo-macos
      - run:
          # build our application
          name: Build Application
          command: xcodebuild
      - run:
          # compress Xcode's build output so that it can be stored as an artifact
          name: Compress app for storage
          command: zip -r app.zip build/Release/circleci-demo-macos.app
      - store_artifacts: # store this build output. Read more: https://circleci.com/docs/2.0/artifacts/
          path: app.zip
          destination: app
```


## Next Steps

The macOS executor is commonly used for testing and building iOS applications,
which can be more complex in their continuous integrations configuration. If you
are interested in building and/or testing iOS applications, consider checking
out our following docs that further explore this topic:

- [Testing iOS Applications on macOS]({{ site.baseurl }}/2.0/testing-ios)
- [iOS Project Tutorial]({{ site.baseurl }}/2.0/ios-tutorial)
- [Setting Up Code Signing for iOS Projects]({{ site.baseurl }}/2.0/ios-codesigning)




...
