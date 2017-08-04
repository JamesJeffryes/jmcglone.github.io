---
layout: post
title:  Tracking custom issues to evaluate commit quality
date:   2017-08-04
categories: Redis, GitHub, Code Quality
---

One of the real strengths of Github is the collection of tools which are integrated into repositories though webhooks. One of the most valuable is the ability to automatically run unit tests on each and every commit with services like TravisCI. Just like GitHub itself isn’t limited to code but can be used for any collaborative data project, Travis doesn’t have to be use for just traditional code tests. Maybe you are like [Paul Romer](https://www.ft.com/content/e706bc4a-4168-11e7-9d56-25f963e998b2) and want to make sure that your repo’s docs are not overusing the conjunction “and”. No problem. You can write a test for that but the output is binary: it hits the target or it fails. But what if you are making commits that improve the issue? Should your builds fail until you hit the target?

I encountered this issue when working to improve a biochemistry database stored on GitHub. It may seem a strange use of git, but actually repositories are in many ways for storing and sharing scientific data; it's open, clearly versioned and supports collaborative editing. In this case, automatic testing with TravisCI is critical to making sure that additions to the database are high-quality and consistent with the existing data. I was able to automate testing to identify duplicates, dead references and inconsistencies in the data. The problem is, these problems already existed in the database so every build would fail until the database was perfect. What I needed was a way to determine if the commit took the database in the right direction. Did it fix more errors than it created?

My inspiration for this automatic code quality checking services like [Codacy](https://www.codacy.com/) or [Code Climate](https://codeclimate.com/), which examine code for coding style violations and evaluate commits on the basis of if they reduce or increase technical debt. I’d love to use these tools (and their pretty dashboards) but they are tightly integrated with the linters they use for error checking and not very amicable to writing custom checks, especially ones with a dependency on a Cheminformaics package. So I am left implementing the critical database functionality myself.

I want this functionality to be distributed just like the tests run by TravisCI tests that would be using it. I settled on Redis, a key-value as database to store any issues associated with a commit based on it’s simplicity and the existence of several cloud based implementations with free tiers. 30MB isn’t much but enough to store error types an counts for commits to the repo for the foreseeable future. I coded up the wrapper for the reds instance as a python module and registered it on PiPy. Now the module can be installed in any .travis.yml file I like and integrated into my tests in just a handful of lines. Now my build will only break if issues are present in the current branch that are NOT present at the head of the master branch!

```
> from repostat.stash import StatStash
> stash = StatStash(<web_endpoint>)
> issues = run_your_test_library()
> new_issues = stash.get_new('example errors', issues)
> stash.report_stats('example errors', issues) # make sure the record is updated regardless
> if new_issues:
    print("New issues detected: %s" % new_issues)
    exit(1) # causes travis build to fail
```

You can check out the source at (GitHub)[https://github.com/JamesJeffryes/repo-stat-stash] or `pip install repostash` to try it out yourself