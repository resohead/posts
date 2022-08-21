---
title: Queue cheatsheet
slug: queue-cheatsheet
description: Reference Laravel queue commands and configuration options.
date: 2022-08-15
tags: [php, laravel, snippet, queues, cache, cheatsheet]
sources:
    - https://ryangjchandler.co.uk/posts/all-about-match-expressions
---

# Queues

Process background jobs with a number of configuration options.

* queue name = an identifier to help prioritise groups of jobs, `php artisan queue:work --queue=payments, default`
* connection = queue service on SQS, Beanstalk, Redis, database etc that can have many queues
* tries = number of failed attempts (exceptions, releases) a worker allows until the job is considered a failure
* automatic retries = see `tries`
* backoff = time between failed job retries from uncaught exceptions - can be an integer or array of integers
* retryAfter = deprecated name for backoff in Laravel versions `< 8.x`
* timeout = how long the job can run for before considered a failure
* workers = number of processes running on the backlog
* failed jobs = a maintenance table to inspect failed jobs - push back onto the queue using `php artisan queue:retry`
* release = a manual delay that overrides the `backoff` value
* maxExceptions = limit the number of tries when failures occur due to exceptions
* job chain = a list of jobs to run in sequence after the previous finishes (if one job fails the entire chain is removed from the queue) beware: the previous job did run
* job batch = a parallel worker job chain. if one item fails the entire batch is considered a failure unless `allowFailures()` is used
* job batch chain = an array within a job batch that runs each in parallel but each set of jobs within the chain in sequence
* job batch catch = a hook into any job failure within a batch.
* job batch then = a hook into a successfully completed batch (all jobs processed without failure)
* job batch finally = a hook into a finished batch (regardless if successful)
* race condition = a situation where two or more processes try to make changes on the same resource at the same time
* atomic locks = a named cache method that blocks a resource from further changes
* funnel = a Redis option to obtain atomic locks with more control
* throttle = a Redis option to allow a number of locks within a timeframe
* without overlapping = a middleware using a key/name that keeps releasing a job if another job with the same key is in progress
* unique = an interface to prevent pushing the same job to the queue multiple times
* throttles exceptions = a circuit breaker around a job if it's failing too often. Allow a number of consecutive failures to prevent any new instances of the job from running. It can prevent overwhelming your system during a third party outage.
* after commit = dispatching jobs from within a database transaction can cause issues when the job attempts to use resources not yet created. job disptach has an after commit method to prevent this per function or you can change the database queue driver config
* self contained = the job has everything it needs to run without relying on any external data. You can do this by passing in the relevant state for that job via the constructor rather than 'calculating' it at run time. A job could be processed later than expected due to failures, backlog, server issues so we don't want 'new' data leaking into an old job.
* encryption = prevent a payload that takes sensitive data from being inspected from the queue sotre (cache/database)
* supervisor = a process monitoring tool that ensures a the queue is always running
* restart signal = workers running old code will finish the current job before being updated before pulling new jobs
* supervisor stop start = `sudo supervisorctl [stop/start] worker-{id}:*` a blocking function that ensures all workers are restarted at the same time to ensure code updates and database migration consistency.
* scaling workers = you can add a `.conf` file to the `/etc/supervisor/conf` directory along with a cron schedule to start and stop workers at regular intervals (know busy periods).

Looking for more than just a cheat sheet? Try [Laravel Queues In Action](https://learn-laravel-queues.com) by Mohamed Said (VP of Engineering at Foodics and former Laravel core team member).