---
title: Debugging an Elastic Beanstalk Deployment Pipeline
subtitle: Mirroring the build step of AWS Elastic Beanstalk locally to debug deployments
tl;dr: Replicate your build process locally without Docker’s cache. Check the resulting Docker image and upload a fresh zip file to EB for deployment to your environment. Run your build pipeline successfully and get back to work.
keywords:
 - AWS
 - Elastic Beanstalk
 - Debugging
 - Docker
date: 1544900219970
updated: null
---

First, some context. Our pipeline consists of a CI build process that is triggered by an update to a GitHub branch. The build process runs some custom commands, creates a zip file for S3, and then deploys that build to Elastic Beanstalk. All this automation is awesome when things are working 🍻, but it can leave you feeling powerless if something goes wrong 😭.

As a start, you should understand that Elastic Beanstalk is just the management layer for the resources your app uses on AWS, so you’re really debugging the underlying resources and your own deployment pipeline. Debugging on remote systems can be a nightmare, especially doing so through a GUI like Elastic Beanstalk where hunting down information can be confusing.

The first thing to do is to replicate as much of your build process locally as you can, taking care to start fresh with every build. There are two big reasons to replicate you build pipeline locally:

1. You need information and the best place to run an experiment is in an environment you have (nearly) total control of
2. Docker will cache a lot of the results from your RUN commands on EB, trapping any build gremlins 👹in a neat little cage, ready to wreak havoc on your next deployment despite the updates you’ve made. The tool you need here is Docker’s --no-cache option so you can flush any nasties right out. Unfortunately this isn’t an option on EB (you can’t specify your build commands) so you will need to run this locally

Here’s a quick guide for creating a clean build:

1. Start with duplicating every command in your CI pipeline: from npm install to your build steps
2. Build a clean Docker image (without the cache) using this command: docker build --no-cache -t app_name .
3. Bundle the Docker image: zip ../version.zip -r * .[^.]*
4. Run your Docker image locally and make sure you can get the thing working

You should now have either found the source of your pain, or have a nice, fresh zip file you can upload to EB.

1. Upload this zip to Elastic Beanstalk in the Application Versions section of your Application Environment.
2. Deploy the uploaded zip to an Application Environment and wait for the build to finish.

If all is well at this point, trigger a new build from your CI (if using one) and with any luck, your deployment pipeline will work and you can go back to focusing on building your awesome features instead of banging your head against the wall 🎉. If your deployment failed, rinse and repeat the above steps, digging deeper into each section to gain more information; you will get there eventually 😬.