+++
title = "SpaghettiHub"
type = "chapter"
weight = 2
+++

I wrote some tools to speed up my productivity. You can find them at [https://spaghettihub.r00ta.com](https://spaghettihub.r00ta.com). Also, [here](https://github.com/r00ta/SpaghettiHub) you can find the github repository for this project. 

Here's a short description about what the tools are doing.

# LaunchHub

You can convert Launchpad Merge proposals to GitHub pull requests. This is mainly because reviewing large merge proposals in launchpad is challenging and because I did some integrations with Github actions to perform some automatic tests. 

The access to this tool is restricted, but you can ask for access to it.

# Continuous delivery status

This tool shows the status of my continuous delivery pipeline for MAAS. This pipeline is executed for every commit on `master` and it's executed for both debs and snaps. This way, you can see if one commit is breaking the build. 

This tools has been developed because multiple times the debs were broken for months and nobody noticed. Finding the root cause was time consuming, so I wrote this tool to avoid this and save some time. 

# Commit to MP search tool

When I have to investigate bugs, I spend hours reading the code and I can't thank enough the people who inveted `git blame`. Sometime the commit message is not enough to understand what's going on, so I wrote a tool to find the merge proposal that landed that specific commit. This is because sometimes in the merge proposal itself there is more context about a change, and in launchpad is very time consuming to find that. 

# AI Bug Finder

This is a tool to find bugs that are semantically similar to a query that you submit. I simply took [https://github.com/jurekh/bug_triage](https://github.com/jurekh/bug_triage) and made it available as a service that everybody can consume. 

