---
layout: classic-docs
title: Migrating From Travis CI
categories: [migration]
description: Migrating from Travi CI
---

This document provides an overview of how to migrate from Travis CI to CircleCI.
We'll step through setting up different features of CI ranging in complexity and use of
features for each system.

The examples that follow will be applied to an [example JavaScript
repository](). For context (and limiting the scope of this article), we will consider the example repository's owner to
want to achieve the following things:

- On pushing code: run a build and run all tests.
- Expect that build time will be reduced by caching any dependencies after the
  initial build.
- Enable the safe use of environment variables.
- On each build, upload a `test-results.xml` file to be made accessible online.
- On pushing a git tag: create a Github Release, and also deploy.

## Pre-requisites

This document assumes that you have an account with CircleCI that is linked
to a repository. If you don't, consider going over our [getting started guide]({{ site.baseurl }}/2.0/getting-started/).

## Configuration Files

Both Travis and CircleCI make use of a _configuration file_ to determine what
each Continuous Integration provider does respectively. With Travis, your
configuration will live in a `.travis.yml` file in the root of your repository.
With CircleCI, your configuration will live in `.circleci/config.yml` at the
root of your repository.

## Building on Pushing Code

Let's look at the minimum viable config we can use to get our build running
before we explore more complex configuration choices.

For the example repository, the beginnings of a simple Travis Configuration might look like so:

```yaml
language: node_js
services: mongodb
before_install: 
  - npm i -g npm@5
node_js:
  - "5"
cache: npm
```

For basic builds, a TravisCI configuration will leverage a language's best known
dependency and build tools and will abstract them away as default commands
(which can be overridden) in the [the job lifecycle](https://docs.travis-ci.com/user/job-lifecycle/#the-job-lifecycle). In this
case, when the build runs, TravisCI will automatically run `npm install` for the
`install` step, and run `npm start` for the `script` step.

If a user needs more control with their CI environment, TravisCI uses _hooks_
to run commands before/after the `install` and `script` steps. In our case, we
want to specify that the npm version we use is pinned to `5`, so we execute the
installation of npm@5 in the `before_install` hook. Hooks can execute shell
scripts as well, which users will sometimes store in a `.travis` folder at the
root of their repository.

A CircleCI configuration looks like so:

{% raw %}
```yaml
version: 2
jobs:
  build:
    working_directory: ~/mern-starter
    docker:
      - image: circleci/node:4.8.2
      - image: mongo:3.4.4
    steps:
      - checkout
      - run:
          name: update-npm
          command: 'sudo npm install -g npm@5'
      - restore_cache:
          key: dependency-cache-{{ checksum "package.json" }}
      - run:
          name: install-npm-wee
          command: npm install
      - save_cache:
          key: dependency-cache-{{ checksum "package.json" }}
          paths:
            - ./node_modules
      - run:
          name: test
          command: npm test
```
{% endraw %}

In the config above, no _language_ is specifically required, and the
user is able to specify any number of `steps` that can be run, with no
restrictions on order or step. By leveraging Docker, specific Node.js and
MongoDB versions are made available in each `command` that gets run.

**On Caching Dependencies**

With CircleCI you have control over when and how your config caches and restore dependencies. In the above example, the CircleCI `.config`
checks for a dependency cache based specifically on a checksum of the
`package.json` file. You can set your cache based on any key as well as set a
group of cache paths to defer to in a specific order. Refer to the [caching
dependencies document]({{ site.baseurl }}/2.0/caching/) to learn about customizing how your build
creates and restores caches for more fine-grained control.

In a Travis Configuration, the [dependency caching](https://docs.travis-ci.com/user/caching/) as a step in your build happens after the
`script` phase of a build, and is tied to the language you are using. In our
case, by using the `cache: npm` key in `.travis.yml`, dependencies will be
cached via `node_modules`.

**On Using Containers**

With CircleCI, the context in which your checked out code executes (builds,
tests, etc) is known as an [Executor]({{ site.baseurl }}/2.0/executor-intro/). 

If you're coming from TravisCI, using Docker will be the closest means to running
a build based on a language. While you can use any docker image in your
`.config`, CircleCI maintains several [Docker Images]({{ site.baseurl
}}/2.0/circleci-images/) tailored for your `.config`.

## Environment Variables

Both Travis and CircleCI enable the use of environment variables in you builds.

In your CircleCI `.config` you can put environment variables directly in your
build config in a step, a job, or a container. Remember,
these variables are public and unencrypted. With TravisCI, it is 
[possible](https://docs.travis-ci.com/user/environment-variables#defining-encrypted-variables-in-travisyml)
to include encrypted environment variables directly in your config if you
install the `travis` gem). If you have private information in your environment
variables, consider securely setting environment variables in the CircleCI web application.

**Setting Environment Variables in the Web Application**

If you've used TravisCI's [repository
settings](https://docs.travis-ci.com/user/environment-variables#defining-variables-in-repository-settings),
you'll be comfortable setting your environment variables in CircleCI's project
settings page. Read the docs for setting environment variable in a [single
project]({{ site.baseurl }}/2.0/env-vars/#setting-an-environment-variable-in-a-project).

With CircleCI, it is also possible to securely set environment variables across
_all_ projects using [contexts]({{site.baseurl}}/2.0/contexts/).

**Note:** CircleCI has several [built-in environment variables](https://circleci.com/docs/2.0/env-vars/#built-in-environment-variables).

## Test Results / Artifacts

## Build Rules based on Git Tags
