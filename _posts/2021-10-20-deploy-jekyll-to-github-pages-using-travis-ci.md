---
title: How to deploy a jekyll site to GitHub Pages using Travis CI
date: '2021-10-20'
tags: jekyll github github-pages travis ci/cd
categories: [Programming, Deployment]
tags: [programming, github-pages, github, travis, jekyll]
---

As of today, this blog runs on Jekyll, and is hosted on Github Pages(located at https://github.com/AyushG3112/blog), deployed via Travis CI on every commit to the `master` branch. This post explains how to set this up easily.
<!--more-->

## Enable Travis CI for your repository
First thing you need to do is migrate add your Github Repository to Travis and enable it. [This article][1] explains how to do it.

## Prepare your project for Travis

### Make your dependencies Travis Compatible

Your `Gemfile.lock` should be compatible with the `x86_64-linux` achitecture because that is what the Travis Ruby image uses as of the date of writing of this post. If you don't you could end up seeing errors like:

Your bundle only supports platforms ["x86_64-darwin"] but your local platforms are ["ruby", "x86_64-linux"]
{:.error}

To avoid this, run the following command in your project directory:

``` bash
bundle lock --add-platform x86_64-linux
```

### Generate your GitHub Personal Access Token

Login to GitHub and navigate to https://github.com/settings/tokens/new to generate a new Personal Access Token(PAT) with the following scopes:

- repo
    - repo:status
    - repo_deployment
    - public_repo Access
    - repo:invite Access
    - security_events

Now, copy your PAT and add it to the environment variables of your Travis repository by following [these steps][3].

This article assumes you stored your environment variable using the `GH_TOKEN` key. If you chose a different name, change the vsriable name in the `.travis.yml` shown below.
{:.info}

### Fix your ruby version

Next, we need to specify the Ruby version which Travis will use to build your site and install dependencies. Since the latest jekyll release(`v4.2.1`) works completely fine with ruby version 2.5.3, that's what we'll be using. So go to your project root and run


``` bash
echo "2.5.3" > .ruby-version
```

### Create your Travis configuration file

Finally, it's time to create your `.travis.yml` file. Here is what I use currently currently with a brief explanation:


``` yaml
language: ruby # build this using ruby

cache: bundler # cache the bundler dependencies to prevent having to reinstall them for each build from scratch

script:
  - rm -rf _site # remove the `_site` folder if it already exists
  - gem install jekyll bundler # install jekyll and bundler
  - bundle install # install all needed gems using bundler
  - JEKYLL_ENV=production bundle exec jekyll build # build the site. This puts the output in the `_site` folder by default
  - find . -mindepth 1 -maxdepth 1 -not -name _site -not -name .git -exec rm -rf '{}' \; # delete all files and folders excluding the `.git` and `_site` folder
  - find _site -mindepth 1 -maxdepth 1 -exec mv '{}' . \; # move all files and folders in the `_site` folder to the root
  - rm -rf _site # delete the site folder
  - echo "example.com" > CNAME # Use `example.com` custom domain for my GitHub Pages site. REMOVE OR UPDATE THIS.

deploy: # deploy step for travis
  provider: pages # to github pages
  skip_cleanup: true
  github_token: $GH_TOKEN # Environment variable key which stores your GitHub PAT
  keep_history: false # do a force push
  on:
    branch: master # only deploy on merges to master branch
```

Make sure to update or remove the `echo "example.com" > CNAME` line based on your custom domain requirements.
{:.warning}

## And thats it!

Any pushes to the `master` branch should now deploy your changes to GitHub Pages on the `gh-pages` branch. Note that the changes might take a while to reflect due to GitHub CDN propogation delays. Be sure check out [my repository][3] to see my integration with Travis CI, Jekyll and GitHub Pages as an example.



[1]: https://docs.travis-ci.com/user/migrate/open-source-repository-migration#migrating-a-repository
[2]: https://docs.travis-ci.com/user/environment-variables/#defining-variables-in-repository-settings
[3]: https://github.com/AyushG3112/blog