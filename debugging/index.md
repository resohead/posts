---
title: Debugging
slug: debugging
description: How to debug anything.
date: 2022-08-16
tags: []
sources: []
---

# Debugging

## What's the issue?
Write down the most basic description of the issue with
- simple language
- short sentences or lists
- no assumptions

For example:
- some cron tasks are not running

### Common features?
Have you received multiple reports of the same issue, or very similar issues?

After writing down the issue above make a separate list of the things that are in common between the issues. It doesn't have to be exhaustive, and you might need to keep refining this as you go along, but it helps spot patterns and making the brain work in the background. For example,

- time of day / month
- system load / high traffic events (e.g. month end for accounting department)
- user roles
- new/old users
- configuration options (user settings)
- feature flags

You aren't looking through code here - just look for patterns. Can you find users/teams that have similar setups that do not experience these records same issue - make a note of this for later!


### Information gathering

A slightly deeper look into the issue where you will make some notes of specific information that you can use to narrow down the issue.

- references (make a note of any affected IDs and timestamps)
- when did the issue start? Check commits on or before this date
- skim the codebase and casually make a note of anything that could potentially have an impact
- known issues on any 3rd party dependencies or APIs?

Manually check the records located around the same time of the issue. Did they experience the same issue but just didn't report it?

If you think the issue is in a well tested area with majority of users not experiencing any issues then you are probably looking at an edge case with a one line (or one character fix)! Keep your eyes peeled.

### Point of failure

This stage is still an information gathering stage, but it's a bit more specific. Using the information from the previous steps try to narrow down the area/module of interest (e.g. cron jobs, race conditions, missing records) by ranking the potential for issues.

You should consider:
- multiple entry points
- issues that overlap with code without any test coverage
- availability of relevant logs

Also look out for:
- look out for complex conditionals (are you following the logic correct? )
- queries that rely on a number of variables
- loops
- events fired (are there any listeners you are missing)

## Make a hypothesis
You have as much information as you think you need at this stage, you have selected the areas of interest and skimmed the codebase.

Think of this like a PR summary but before you have fixed the issue.

For example, in a web monitoring saas: "Users are only sometimes notified that a site has recovered because 'downtime ended' events used a cached value on the site record as well as a downtime log. If these two values fell out of sync records might be updated but users will not be notified".

Then you could also suggest a fix: "We will no longer use the cached value and instead fire an event as a result of any downtime periods being updated. We will lock the row for updates and use database transactions to ensure notifications are fired after successful updates".

## Fix and repeat

If your theory was incorrect and your fix didn't work, go back to the previous steps and try again. This can be frustrating especially after spending time planning and making code changes.

Here's a few more things you can use to help you further.

### Production logging
- send messages with important information
- be selective (use conditionals) to prevent overwhelming you with information
- log info at a single place where different paths could be taken

### Puzzle pieces
- remove code (clutter)
- drop your ego and write in pseudo-code
- if you keep asking the same question without an answer consider external factors such as...
- read it
- read it again

### Indirect causes
- API changes (undocumented and unannounced could be harder to find)
- scheduled tasks (is everything running on time, do your tests take cleanup tasks into account)
- queues / race conditions (does this bug occurr around particularly busy periods or in a write-heavy part of the application)
- timezones (do you have users in different locations, are you storing user's timezone and are you storing dates/times in UTC)
- server setup vs local config (environment variables and dependencies on the same version)

### Amplify
Try changing inputs or local code to extreme or unlikely vales and observe results. This is like a binary search but for debugging to bypass a large chunk of potential issues and force different errors.

For example, you could run your tests with scheduled tasks running and then run without any any scheduled tasks running.

## Take a break
If you have done at least one cycle and got to the point of frustration take a longer break, do some manual work or work on an easy task.
You will come up with a new approach or hypothesis to test but if you're still stuck take a look at the areas you quickly dismissed at the start.

## Write tests
- if you can make a failing test you're 95% there
- unit test for sense checking using multiple inputs (data providers for coverage)
- seed the test data with the same values as the real data

And if don't have any tests it might be a good time to start...it will help you when debugging the next issue as well as give you confidence when refactoring or deploying.

## Summary

It's simple - find the inputs that cause failures without impacting the rest of the application or current users 😂
I said simple, not easy! It's also not always a fast process but keep being stubborn and you might get lucky.