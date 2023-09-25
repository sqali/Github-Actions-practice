# Github-Actions-practice

[![CI](https://github.com/sqali/Github-Actions-practice/actions/workflows/basic.yml/badge.svg)](https://github.com/sqali/Github-Actions-practice/actions/workflows/basic.yml)


# Notes

## 

## Uploading & downloading Artifacts

Artifacts, as Actions defines them, are simply files or collections of files, created as the result of a job or workflow run and persisted in GitHub. The persistence is usually so that the artifact can be shared with other jobs in the same workflow, although you might also persist an artifact to have access to it via the UI or a REST API call after the run was complete.

Examples of artifacts could include modules produced in a build job that then need to be packaged or tested by other jobs. Or you might have log files or test output that you want to persist to look at outside of GitHub. While these items can be persisted, they cannot be persisted indefinitely. By default, GitHub will keep your artifacts (and build logs) around for 90 days for a public repository.

Upload Build artifacts arguments description table


| Name              | Required | Default                                     | Description                                                                                                                                                    |
|-------------------|----------|---------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ```name```             | yes      | artifact                                    | Name of the artifact.                                                                                                                                          |
| ```path```            | yes      | none                                        | File system path to what you want to upload.                                                                                                                   |
| ```if-no-files-found``` | no       | warn                                        | What to do if there are no files in the path you specified: a value of error means stop with an error; a value of warn means report the issue but don’t fail; a value of ignore means don’t fail and don’t print a warning—just keep going. |
| ```retention-days```    | no       | 0 (which means use the repository default; see description) | Number of days before the artifact will expire (be removed from GitHub). Can be between 1 and 90 to indicate a specific number of days or 0 to use the default.  |


## Caches in GitHub Actions

GitHub Actions includes the ability to cache dependencies to make actions, jobs, and workflows more efficient and faster to execute. Common packaging and dependency management tooling such as npm, Yarn, Maven, and Gradle create a cache of their downloaded dependencies (usually under hidden areas such as .m2 for Maven and .gradle for Gradle). The caching that is built into these applications saves time when using the same tooling on the same machine since the dependencies don’t have to be downloaded again.

However, if you’re using runner systems hosted on GitHub, each job in an action starts in a clean execution environment, downloading dependencies again by default. This results in using more network bandwidth and longer runtimes. For paid plans, this can ultimately increase the cost of using actions. To help with these issues, GitHub Actions can cache dependencies used frequently by these applications.

There are two options for enabling the caching functionality with actions. The first option is to use the explicit cache action. The second is activating it within the various setup-* actions, such as the setup-java action shown previously.

The first option requires more configuration but gives you explicit control over the caching. As well, it allows use across a wider set of applications. The second option requires minimal configuration and manages the creation and restoration of the cache automatically—if you are using one of the setup-* actions.

## GitHub Actions Cache System

| Function   | Name                 | Description                                                                                                                                                        |
|------------|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| input      | ```path```          | A list of files/directories/patterns that specify which file system objects to include in the cache and restore from it                                             |
| input      | ```key```      | An explicit key to use for saving and restoring the cache                                                                                                          |
| input      | ```restore-keys```         | An ordered list of keys to check to indicate that a match occurred with the key; a match here is referred to as a cache hit                                         |
| input      | ```upload-chunk-size```    | Chunk size used to split up large files during upload, in bytes                                                                                                    |
| input      | ```enableCrossOs​Arch⁠ive``` | Optional boolean—allows Windows runners to save or restore caches that can be restored or saved respectively on other platforms                                   |
| input      | ```fail-on-cache-miss```   | Fail workflow if cache entry is not found                                                                                                                         |
| input      | ```lookup-only```          | See if a cache entry exists for the given input(s) (key, restore-keys) without downloading the cache                                                               |
| output     | ```cache-hit```           | Simple boolean to indicate if an exact match is found for a key                                

## Advanced Triggering Workflows
There’s the on keyword introducing the section of events that trigger running this workflow and then the issues trigger below that. It may look a bit strange to have a trigger with simply an ending colon and nothing after that, but it is valid syntax. The implication of this is that this workflow will be triggered for any and every kind of activity that occurs for an issue, such as creation, updating, or deletion.

If this is what you need, that’s great. But, if you need to refine more when your workflow runs, there are options you can supply for the triggers in the on section. These are referred to as activity types.

## Refining Triggers

Some triggering events allow using filters to further define when a workflow will run in response to the event. A filter is specified using a keyword that defines the type of entity to filter, and one or more strings that are specific names or patterns. The strings can use standard glob syntax (*, **, ?, !, +, etc.) to match multiples.

A good example is qualifying which branches and tags cause a workflow to run when a push event occurs. You can filter a list of branches and tags for the push event with wildcards for pattern matching:

You can also specify a set of branches to exclude via the keyword branches-ignore, tags to exclude via tags-ignore, or paths to exclude via paths-ignore. The use case for this is when it’s easier or more desirable to specify a set of branches or tags in your repository not to trigger off of rather than a set to trigger off of.

You can’t include both the inclusive and exclusive keywords for the same event (for example, you can’t include both branches and branches-ignore for a push trigger).

You can use the below to ignore certain branches and tags when triggers happen

```
branches-ignore:
    - main
    - 'prod/*'
tags-ignore:
    - 'rc*'
paths-ignore:
    - 'data/**' # will not get triggered if anything is changed within the data subdirectory
```

```
on:
  push:
    paths:
      - 'module1/**'
      - '!module1/data/**'

if anything is changed within module1 then it will be triggered but if the change is also within module1/data then it will not triggered as it has been specified
```

If you’re not familiar with the meaning of the ** symbol, in glob syntax, it matches filenames and directories recursively. Essentially, it matches on anything in a tree structure under the specified path.

If you are triggering off of a push or pull request event, you can refine those more to only trigger off of changes to particular file paths

## Triggering workflows without a change

```workflow_dispatch```: This event allows you to manually trigger a workflow from the GitHub Actions tab. You can provide input parameters and run the workflow on-demand. It doesn't automatically run; you have to initiate it manually.

```workflow_call```: This event allows you to call a reusable workflow from another workflow. You can specify input parameters, and the called workflow runs as part of the calling workflow. It can be used when you want to reuse a set of workflow steps across different workflows.

```workflow_run```: This event allows you to trigger the run of one workflow based on another workflow's execution. You can specify conditions such as which workflow to trigger (workflows), when to trigger it (types), and on which branches or conditions it should run. For example, you can set it up so that when a workflow named "Pipeline" completes, it triggers another workflow to perform additional tasks, such as deployment.
Here's an example scenario for workflow_run:

```
on:
  workflow_run:
    workflows: ["Pipeline"]
    types: [completed]
    branches:
      - 'rel/**'
      - '!rel/**-preprod'
```

In this example, when a workflow named "Pipeline" completes (either successfully or with a failure), it triggers the defined workflow to run. It only runs on branches starting with "rel" but excludes branches ending with "-preprod."

The completed status here means that the pipeline ran to completion, which could be either success or failure. Alternatively, you can also use a status of requested, which implies only that the other workflow has been triggered. In that use case, this workflow would run at effectively the same time as the other one.

Note that there is also a requested status for the workflow_run event. That allows you to sequence execution of workflows in a similar manner, as the needs keyword allows you to sequence execution of jobs within a workflow.

repository_dispatch: This event allows external events to trigger workflows in your repository. You can set up custom events and payloads that can be used by external systems or scripts to trigger specific workflows. It's useful when you want to automate workflows based on events outside of GitHub. For example, you could trigger a deployment workflow when a specific event occurs in an external CI/CD system.

Here's a high-level scenario for repository_dispatch:

You have a CI/CD system external to GitHub.
This external system detects a new version of your software.
The CI/CD system sends a custom "release" event to your GitHub repository using the repository_dispatch API.
GitHub triggers a specific workflow in your repository, such as a deployment workflow, in response to the custom "release" event.
These events give you flexibility in automating workflows in response to various conditions, whether it's manual initiation (workflow_dispatch), reusing workflows in other workflows (workflow_call), or responding to external events (workflow_run and repository_dispatch).


## Dealing with Concurrency

Imagine you have a repository where you use GitHub Actions for Continuous Deployment (CD) to automatically deploy your web application to a production server whenever changes are pushed to the main branch. You want to ensure that only one deployment is happening at a time to avoid conflicts and overloading your production server.

This workflow triggers whenever changes are pushed to the main branch.

In the deploy job, we use the concurrency keyword with the group name "production-deployment." This ensures that only one instance of the deployment job with this group name can run at a time.

We set cancel-in-progress: true to make sure that if a new deployment is triggered while an existing one is still running (e.g., due to rapid successive pushes), the new deployment cancels the previous one to avoid conflicts.

Inside the deploy job, we have steps to check out the code from the repository and then run the deployment script, which deploys the application to the production server. We use secrets to securely store sensitive information like server credentials.

Consider this scenario:

Developer A pushes a change to the main branch, triggering the CD workflow.
Simultaneously, Developer B pushes another change to the main branch.
Without the concurrency setting, both Developer A and Developer B's workflow instances would start concurrently, and both might try to deploy to the production server simultaneously, which could lead to conflicts and issues.

With the concurrency setting:

When Developer A's workflow starts, it will create a "lock" in the "production-deployment" concurrency group. This means no other workflow instance with the same concurrency group can start the deploy job until Developer A's workflow completes.

If Developer B's workflow starts while Developer A's workflow is still running, it checks the concurrency group and sees that an instance with the same group name is already running (Developer A's workflow). In this case, Developer B's workflow will wait or be canceled, depending on the cancel-in-progress setting.

Regarding the concept of the "group" in concurrency:

The "group" is simply a label or identifier that you assign to a concurrency setting. It can be any string or expression you choose.

When multiple jobs or workflows use the same concurrency group, they effectively compete for access to resources. Only one job or workflow with the same concurrency group can run at a time.

You use this group name to group related jobs or workflows that should not run concurrently. In the example, "production-deployment" is used to group all instances of the deploy job within the same workflow.

It's a way to coordinate and control access to shared resources, ensuring that potentially conflicting actions don't happen simultaneously.

So, by assigning a common "group" name to related jobs or workflows, you can ensure that only one instance of those jobs or workflows runs at a time, even if multiple instances of the same workflow are triggered simultaneously.

```
jobs:
  build:

    runs-on: ubuntu-latest
    
    concurrency:
      group: ${{ github.ref }}
      cancel-in-progress: true
```

To ensure that your concurrency group is unique to your workflow (to avoid cancelling other workflows in the same repository unintentionally), you can add the workflow property from the github context to it, as follows:

```
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

USING UNDEFINED CONTEXT VALUES IN CONCURRENCY GROUPS
Be aware that since some context values are only defined for certain types of events, if you use them in concurrency group specifications and the triggering event does not provide that value, you will end up with a syntax error. For example, assume I have the following code:

```
on:
  push:      
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
        
concurrency:
  group: ${{ github.head_ref }}
  cancel-in-progress: true
```   

Since the head_ref property of the github context is only defined when a pull request is done, if I did a push with this code, I would get a syntax error because the head_ref property is undefined on a push.

Here’s an example of such an error:

The workflow is not valid. .github/workflows/simple-pipe.yml (Line: 22,
Col: 14): Unexpected value ''
To prevent this, you can use a logical OR operation to have a fallback:

```
concurrency:
  group: ${{ github.head_ref || github.ref }}
  cancel-in-progress: true
```

## Running a workflow with a matrix

Sometimes, you may need to execute the same workflow multiple times based on different dimensions of data. For example, perhaps you need to run the same test cases across multiple browsers. Or you need to run the test cases across multiple browsers on each of multiple operating systems. For these kinds of cases, you can leverage the matrix strategy within GitHub Actions. You specify this strategy for the jobs in your workflow and define a matrix of dimensions you want to execute across. GitHub Actions will then generate jobs for each combination and execute them accordingly.

The continue-on-error setting for jobs and steps can be used with the matrix strategy to allow the matrix processing to continue iterating through the combinations defined for your matrix. When this is specified, if one of the combinations fails, this will allow the workflow to continue processing the rest of the matrix.

matrix can take multiple arguments such as below example of code:

```
name: Create demo issue 3

on:
  push:

jobs: 
  create-new-issue:
    strategy:
      matrix:
        prod: [prod1, prod2]
        level: [dev, stage, prod]
    uses: rndrepos/common/.github/workflows/create-issue.yml@v1
    secrets: inherit
    with:
      title: "${{ matrix.prod}} issue"
      body: "Update for ${{ matrix.level}}"
  report-issue-number:
    runs-on: ubuntu-latest
    needs: create-new-issue
    steps:
      - run: echo ${{ needs.create-new-issue.outputs.issue-num }}

```

## Workflow Functions

There are a number of functions built in for use in workflows. Some of them provide convenient ways to inspect, format, or transform strings or other values.

| Function   | Purpose                                                                  | Usage                                     |
|------------|--------------------------------------------------------------------------|-------------------------------------------|
| contains   | Checks if item is contained in a string or array. Return true if found. | `contains(search, item)`                 |
| startsWith | Checks if a string starts with a particular value.                       | `startsWith(searchString, searchValue)`   |
| endsWith   | Checks if a string ends with a particular value.                         | `endsWith(searchString, searchValue)`     |
| format     | Within a given string, replaces occurrences of {0}, {1}, {2}, etc. with the replacement values in the given order. | `format(string, replaceValue0, replaceValue1, ..., replaceValueN)` |
| join       | Concatenates values in the array together into a string; uses comma as the default separator, but a different separator can be specified. | `join(array, optionalSeparator)`         |
| toJSON     | Pretty prints the specified value in JSON format.                       | `toJSON(value)`                           |
| fromJSON   | Returns a JSON object or JSON datatype from the given value; useful to convert env variables from a string to another data type (such as boolean or integer) if needed. | `fromJSON(value)`                         |
| hashFiles  | Returns a hash for the set of files that match the path specified.       | `hashFiles(path)`                         |

## Conditionals and Status Functions

You can use an if clause at the start of a job or a step to check a condition and determine if execution should occur or not. This can be done in a couple of ways.

You can check if context values are related to specific values. For example, the following code checks to see if the event is occurring on the main branch before allowing the job to execute. It also checks the value of the os property for the runner and reports information as part of a step:

```
name: Example workflow

on:
  push:

jobs:
    
  report:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/test'
    steps:
      - name: check-os
        if: runner.os != 'Windows'
        run: echo "The runner's operating system is $RUNNER_OS."
```

Also available are a set of status functions that can be used with conditionals to determine whether to execute a job or a step. Table 8-2 shows the status functions along with explanations and example usage.

| Function   | Meaning                                                                       |
|------------|-------------------------------------------------------------------------------|
| success()  | Returns true when none of the previous steps have been failed or cancelled  |
| always()   | Returns true and always proceeds even if the workflow has been cancelled    |
| cancelled()| Returns true if the workflow was cancelled                                  |
| failure()  | When used with steps, returns true if a previous step failed; when used with jobs, returns true if a previous ancestor job (one that was in the dependency path) failed |


The syntax for these is fairly straightforward. You can write them as if: ${{ success() }}, but you can also use the simpler form of if: success(). And you can combine them with logical operators. An example of this is shown here:

```
create-issue-on-failure:
    
    permissions:
      issues: write
    needs: [test-run, count-args]
    if: always() && failure()
    uses: ./.github/workflows/create-failure-issue.yml 
In this case, I am always checking for a failure, so I can create a GitHub issue to document the failure condition.
```

DON’T ALWAYS RELY ON ALWAYS()
In situations where you want to run a job or step regardless of success or failure and the potential for a critical failure may occur, it is best not to rely on always(). This is because you may end up waiting for a timeout to occur. The recommended approach for this situation is to use if: success() || failure() instead.

Finally, there is also a timeout-minutes setting that can be used to specify the maximum number of minutes that a job should be allowed to run before cancelling it. The default is 360.

## Actions & Security

