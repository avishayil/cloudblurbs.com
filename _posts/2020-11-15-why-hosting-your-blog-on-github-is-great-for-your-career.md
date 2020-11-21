---
layout: post
title: "Why Hosting your Blog on GitHub Pages is Great for your Career"
description: "Why Hosting your Blog on GitHub Pages is Great for your Career"
thumb_image: "github-pages.png"
tags: [github, career, jekyll, blog]
---

{% include image.html path="github-pages.png" path-detail="github-pages.png" alt="GitHub Pages" %}

## Introduction ##

If you haven't started writing your own blog until now, this is the time to start doing it. Blogging is a great way to show the world your way of thinking, and enhance some important soft skills like communication, open-mindedness, and creativity.

It also forces you to deeply understand about the stuff you write, because on your workplace you might convince the others that you're right, but the internet is full of criticism and interests. That's why you should carefully research before you put anything on the paper.

On this blog post, I'm going to share my experience from refreshing my blog as GitHub Pages website, as it was a short but satisfying experience.

### You are Contributing to Open Source ###

Let's assume that your are reading this post as a junior developer, and you're doing your first steps in your career. You can read a lot on the internet, about how important for employers to see practical projects. I can share from my personal experience, as a recruiting manager and as a colleague developer in my past, that employers and appreciating footprint on the open-source world.

Putting some experimental/mature projects on GitHub, answering questions on Stack Overflow, sharing from your experience and ideas on blog posts... It is a good chance to show practical experience, code quality and approaching issues, without even working for a "real" company. And if you find the time to do these side projects on your free time while working in a full time job - This is even more appreciated.

### You are Running a Serverless Application ###

Before I launched this version of the blog, it was hosted on the [WordPress](https://wordpress.org) platform. WordPress platform is great, but it required me to keep EC2 instances on AWS running the blog web application. Keeping all those instances of Apache, PHP, MySQL, Memcached... this incurred costs and maintenance, which is also a way to learn something about maintaining the lifecycle of such projects, but they was stateful. And since I began working on Serverless solutions on my job, I began to think what I can do in order to work on my blog, in a serverless way.

In this blog, I'm using GitHub Pages, which provides to me the platform to host my static website without maintaining any servers, or even S3 buckets configured for static web hosting. GitHub does it all for me, and all I need to do is to write content, commit it and push to my repository. If it passes all the tests, I have the excellent `pull request` platform to hand my post for both machine with [prosebot](https://github.com/JasonEtco/prosebot) and human review, and apply fixes accordingly until I decide to publish it, pretty powerful.

Here's some feedback I got from spelling and good-writing bots:

{% include image.html path="prosebot-checks.png" path-detail="prosebot-checks.png" alt="Prosebot Checks" %}

### You are Working with Different Technologies ###

This blog is built with `Jekyll` engine, which is a static pages generator. Once you start working with it, you can't stop because it's so cool. It is written with `ruby`, so you have a chance to expose yourself to `ruby` scripts and understand a little bit about this syntax of this language. During the work on this blog, I even modified some scripts to fit my own needs, even though I didn't have any experience with `ruby` (I just treated it the same as `python`).

On top of that, `chalk` theme that this blog is built on, using `node` technology to consume some libraries that provides the static website some cool capabilities, like `jquery`, `webfontloader`, `retinajs` which their names are pretty self-explanatory. Also, it uses the `npm scripts` capability to server the blog locally and publish the latest version to GitHub Pages. This forces to install dependencies from `npm` and see how it affects the blog - again, very helpful knowledge.

### You are Building a CI/CD Pipeline ###

In order to automate my publishing process, I used `circleci` as automation platform that is free for open source projects. The pipeline process includes tests - static code analysis for HTML using `HTMLProofer` for checking broken links and HTML syntax errors. If a test fails, You'll get a notification to my email and You'll not be able to publish new posts until the problem had been fixed. Once the tests passed, You'll merge the post branch to the master and the the deploy process will push the latest changes to the `gh-pages` branch.

CircleCI platform also have a nice set of features, and you can learn how to generate and store your GitHub deploy key securely on their platform, and use it on the pipeline in order to push the latest version of the `gh-pages` branch automatically. You can also use their caching platform in order to cache `yarn` and `bundler` dependencies, and shorten the test and deploy time significantly - from 3-4 minutes to ~30 seconds only.

### Markdown Training is Great for your Documentation Skills ###

One of the most underrated skills for developers is the ability to document. Developers are usually tired of documentation, and it can become frustrating to consume other's code without proper instructions. Since you're writing posts using `markdown`, you will find out that soon enough you will enjoy writing documentation, and you will do it way faster than you're used to.

**I hope that after reading this post, I could encourage you to clone my repository and open your own blog hosted on GitHub Pages.**

Keep on writing!

Avishay.
