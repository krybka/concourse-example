Concourse CI pipeline example
=============================

Introduction
------------

The repository fulfills the desire to quickly create a production-ready
Concourse CI pipeline. A [pipeline](https://concourse-ci.org/pipelines.html)
combines [jobs](https://concourse-ci.org/jobs.html) and
[resources](https://concourse-ci.org/resources.html).
A job is a sequence of steps. During the job, you can fetch and update
[resources](https://concourse-ci.org/resources.html) or run
[tasks](https://concourse-ci.org/tasks.html). The tasks can be reused between
jobs and pipelines. Moreover, a pipelines configuration can be parameterized
by [vars](https://concourse-ci.org/vars.html), allowing the configuration to be
submitted with different parameters separated from it.

You can find a set of useful tasks in `tasks` directory. At the beginning 
and at the end of each job it is worth updating the build status - for the commit
in Bitbucket Cloud in this case. Sent build descriptions are defined in
`build_descriptions` directory. Credentials are stored in vars - the files in
`credentials` directory you can threat as templates that need to be filled by
your values.

If you need to immerse yourself in Concourse CI work, you can take a tour of the
[Concourse CI examples](https://concourse-ci.org/examples.html) on the project
site.

Pipelines
---------

There are two pipelines configurations prepared. `build-test-stage-prod.yml`
with all valuable jobs, `build-stage-prod.yml` - a subset of it without test
environment, and `stage-prod` -  a subset of jobs from the second one
without building production like images with the source code.

Not all tasks may be useful in your project - some of them are present to show
the general idea where to place extra actions. So it is very usual to perform
some tasks after the deployment (`init-mongo-rs`) or in order to check the
stability of the production environment (`proxy-check` or `parser-check`).

Jobs in `pipelines/build-test-stage-prod.yml`:

* build production-like images with the source code
* deploy services on the test environment
* run tests in the test environment
* deploy services on the stage environment
* deploy services on the prod environment

If there are no tests in your project, you will not need to set up the test
environment.
Then you can use simplified `pipelines/build-stage-prod.sh` where there are
jobs:

* build production-like images with the source code
* deploy services on the stage environment
* deploy services on the prod environment

Moreover, if you want to start the project composed of public Docker images
only, you will not need to build images with the source code at the beginning.
Then you can use very simplified `pipelines/stage-prod` where there are jobs:

* deploy services on the stage environment
* deploy services on the prod environment

Credentials
-----------

For each pipeline from `pipelines` directory, there is a vars file template in
`credentials` directory. Before setting a new pipeline you should prepare a new
credentials file. As well each project credentials (like passwords) you can
store in a separated and dedicated file with Concourse vars.
Nevertheless, some of them should be prepared and provided by your organization
and generally can be shared between projects:

* `private-repo-key`
* `bitbucker-client-id`
* `bitbucker-client-secret`
* `prod-base-registry-user` (if you build the images)
* `prod-base-registry-pass` (if you build the images)
* `prod-registry-user` (if you build the images)
* `prod-registry-pass` (if you build the images)
* `stage-mongo-replica-keyfile-url` (if you have a `mongo` service) - one per
  project
* `prod-mongo-replica-keyfile-url` (if you have a `mongo` service) - one per
  project

The rest of then you can easily set on your own.

`fly` CLI
---------

You can manage Concourse CI from the command line using the [`fly` CLI tool](
https://concourse-ci.org/fly.html).
You can find download links in the bottom right corner of the main page
of Concourse installation.

First of all, you need to login to a proper target and team:
```
fly --target [target-name] login --concourse-url https://[concourse-url] --team-name [team-name] --open-browser
```

**Remember to create your own `credentials/[pipeline-name].yml` from
`credentials/[pipeline-name].template.yml` which should not be tracked in
the git repository.**

Then you can submit a pipeline configuration to Concourse:
```
fly --target [target-name] set-pipeline --pipeline [pipeline-name] --config ./pipelines/[pipeline-name].yml --load-vars-from credentials/[pipeline-name].yml
```
and unpause it on the pipeline webpage.
