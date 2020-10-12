---
title: "About"
date: 2020-10-07T13:13:16Z
draft: false
toc: true
---

## Me

I've worked within the technology industry for approximately 20 years now (wow - feeling old!).  I'm currently working at Red Hat as a Senior Architect within the consulting team, assisting customers delivering complex IT solutions.  I predominantly work in the storage and container ecosystems but have an interest in most things Linux related.

Prior to this I worked in the hosting industry delivering complex solutions on a global scale.

The purpose of this site is for me to write about things I've found interesting, either via working on them for clients or general interest.

Opinions expressed are my own and not related to/endorsed by my employeer in any fashion.

## This Website

This website is entirely a static website.  It's stored in its entirity within [GitHub](https://github.com/jameswilkins/blog). I utilise [Hugo](https://gohugo.io) to generate a static site from the files stored within git.

Those generated site is then hosted by [NetLify](https://netlify.com) on a global CDN network.

Updates are made by editing on my local workstation and commiting into git.

### Infrastructure

#### Cloudflare

I utilise [Cloudflare](https://cloudflare.com) to both host the DNS and initial ingress point for the website.

SSL is terminated at CloudFlare and then re-encrypted back onto NetLify.

I opted for this approach versus hosting everything with NetLify for a few reasons;

* I like to keep my DNS all in one place as I have other domain names
* I admire CloudFlare's approach to enabling new technologies (for example, HTTP/3) rapidly
* CloudFlare performs a series of optimisations on the website to ensure it load's quickly

#### NetLify

NetLify performs the site construction and publishes the website to their world-wide CDN network.

The site on NetLify has a custom SSL certificate (CloudFlare Origin certificate) which is used to encrypt traffic between CloudFlare <=> NetLify

#### Github

The main site is hosted in GitHub [here](https://github.com/jameswilkins/blog).  

I also make use of [GitHub LFS](https://docs.github.com/en/free-pro-team@latest/github/managing-large-files/working-with-large-files) for specific directories that contain images/other-specific content.  This is to prevent the main repo filling up with (non-necesary) versioning information for these data-types.

#### Workflow

Posting updates to the web-site works like so

* Checkout github repository 
* Update content / create new post
* Commit into github
* NetLify triggers a build once git is updated - it checks out the content from git, applies the hugo generator to the site and then publishes it on a specific URL.  A sample deployment is below

```
3:36:43 PM: Build ready to start
3:37:06 PM: build-image version: b0258b965567defc4a2d7e2f2dec2e00c8f73ad6
3:37:06 PM: build-image tag: v3.4.1
3:37:06 PM: buildbot version: c6376102eedf4be6c6e5d685c7141e2eb612d47d
3:37:06 PM: Fetching cached dependencies
3:37:06 PM: Starting to download cache of 141.2MB
3:37:08 PM: Finished downloading cache in 1.764583414s
3:37:08 PM: Starting to extract cache
3:37:11 PM: Finished extracting cache in 3.085211686s
3:37:11 PM: Finished fetching cache in 4.893654924s
3:37:11 PM: Starting to prepare the repo for build
3:37:11 PM: Netlify Large Media is enabled, running git commands with GIT_LFS_SKIP_SMUDGE=1
3:37:12 PM: Preparing Git Reference refs/heads/main
3:37:14 PM: Starting build script
3:37:14 PM: Installing dependencies
3:37:14 PM: Python version set to 2.7
3:37:15 PM: Started restoring cached node version
3:37:17 PM: Finished restoring cached node version
3:37:18 PM: v12.18.0 is already installed.
3:37:19 PM: Now using node v12.18.0 (npm v6.14.4)
3:37:19 PM: Started restoring cached build plugins
3:37:19 PM: Finished restoring cached build plugins
3:37:19 PM: Attempting ruby version 2.7.1, read from environment
3:37:20 PM: Using ruby version 2.7.1
3:37:20 PM: Using PHP version 5.6
3:37:20 PM: 5.2 is already installed.
3:37:20 PM: Using Swift version 5.2
3:37:20 PM: Installing Hugo 0.70.0
3:37:20 PM: Hugo Static Site Generator v0.70.0-7F47B99E/extended linux/amd64 BuildDate: 2020-05-06T11:26:13Z
3:37:20 PM: Started restoring cached go cache
3:37:20 PM: Finished restoring cached go cache
3:37:20 PM: go version go1.14.4 linux/amd64
3:37:20 PM: go version go1.14.4 linux/amd64
3:37:20 PM: Installing missing commands
3:37:20 PM: Verify run directory
3:37:22 PM: ​
3:37:22 PM: ┌─────────────────────────────┐
3:37:22 PM: │        Netlify Build        │
3:37:22 PM: └─────────────────────────────┘
3:37:22 PM: ​
3:37:22 PM: ❯ Version
3:37:22 PM:   @netlify/build 4.8.1
3:37:22 PM: ​
3:37:22 PM: ❯ Flags
3:37:22 PM:   deployId: 5f7dd27aff9a570007efa9dd
3:37:22 PM:   mode: buildbot
3:37:22 PM: ​
3:37:22 PM: ❯ Current directory
3:37:22 PM:   /opt/build/repo
3:37:22 PM: ​
3:37:22 PM: ❯ Config file
3:37:22 PM:   /opt/build/repo/netlify.toml
3:37:22 PM: ​
3:37:22 PM: ❯ Context
3:37:22 PM:   production
3:37:22 PM: ​
3:37:22 PM: ┌────────────────────────────────────┐
3:37:22 PM: │ 1. build.command from netlify.toml │
3:37:22 PM: └────────────────────────────────────┘
3:37:22 PM: ​
3:37:22 PM: $ hugo --gc --minify
3:37:22 PM: Building sites …
3:37:22 PM:                    | EN
3:37:22 PM: -------------------+-----
3:37:22 PM:   Pages            | 13
3:37:22 PM:   Paginator pages  |  0
3:37:22 PM:   Non-page files   |  0
3:37:22 PM:   Static files     | 85
3:37:22 PM:   Processed images |  0
3:37:22 PM:   Aliases          |  3
3:37:22 PM:   Sitemaps         |  1
3:37:22 PM:   Cleaned          |  0
3:37:22 PM: Total in 180 ms
3:37:22 PM: ​
3:37:22 PM: (build.command completed in 233ms)
3:37:22 PM: ​
3:37:22 PM: ┌─────────────────────────────┐
3:37:22 PM: │   Netlify Build Complete    │
3:37:22 PM: └─────────────────────────────┘
3:37:22 PM: ​
3:37:22 PM: (Netlify Build completed in 268ms)
3:37:22 PM: Caching artifacts
3:37:22 PM: Started saving build plugins
3:37:22 PM: Finished saving build plugins
3:37:22 PM: Started saving pip cache
3:37:22 PM: Finished saving pip cache
3:37:22 PM: Started saving emacs cask dependencies
3:37:22 PM: Finished saving emacs cask dependencies
3:37:22 PM: Started saving maven dependencies
3:37:22 PM: Finished saving maven dependencies
3:37:22 PM: Started saving boot dependencies
3:37:22 PM: Finished saving boot dependencies
3:37:22 PM: Started saving go dependencies
3:37:22 PM: Finished saving go dependencies
3:37:22 PM: Build script success
3:37:22 PM: Starting to deploy site from 'public'
3:37:22 PM: Creating deploy tree 
3:37:22 PM: Creating deploy upload records
3:37:23 PM: 9 new files to upload
3:37:23 PM: 0 new functions to upload
3:37:24 PM: Starting post processing
3:37:24 PM: Post processing - HTML
3:37:24 PM: Post processing - header rules
3:37:24 PM: Post processing - redirect rules
3:37:24 PM: Post processing done
3:37:25 PM: Site is live
3:37:43 PM: Finished processing build request in 37.637073072s
```

* At this point, the updated website is available




