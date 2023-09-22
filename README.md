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
| name              | yes      | artifact                                    | Name of the artifact.                                                                                                                                          |
| path              | yes      | none                                        | File system path to what you want to upload.                                                                                                                   |
| if-no-files-found | no       | warn                                        | What to do if there are no files in the path you specified: a value of error means stop with an error; a value of warn means report the issue but don’t fail; a value of ignore means don’t fail and don’t print a warning—just keep going. |
| retention-days    | no       | 0 (which means use the repository default; see description) | Number of days before the artifact will expire (be removed from GitHub). Can be between 1 and 90 to indicate a specific number of days or 0 to use the default.  |


## Caches in GitHub Actions

GitHub Actions includes the ability to cache dependencies to make actions, jobs, and workflows more efficient and faster to execute. Common packaging and dependency management tooling such as npm, Yarn, Maven, and Gradle create a cache of their downloaded dependencies (usually under hidden areas such as .m2 for Maven and .gradle for Gradle). The caching that is built into these applications saves time when using the same tooling on the same machine since the dependencies don’t have to be downloaded again.

However, if you’re using runner systems hosted on GitHub, each job in an action starts in a clean execution environment, downloading dependencies again by default. This results in using more network bandwidth and longer runtimes. For paid plans, this can ultimately increase the cost of using actions. To help with these issues, GitHub Actions can cache dependencies used frequently by these applications.

There are two options for enabling the caching functionality with actions. The first option is to use the explicit cache action. The second is activating it within the various setup-* actions, such as the setup-java action shown previously.

The first option requires more configuration but gives you explicit control over the caching. As well, it allows use across a wider set of applications. The second option requires minimal configuration and manages the creation and restoration of the cache automatically—if you are using one of the setup-* actions.

## GitHub Actions Cache System

| Function   | Name                 | Description                                                                                                                                                        |
|------------|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| input      | path                 | A list of files/directories/patterns that specify which file system objects to include in the cache and restore from it                                             |
| input      | key                  | An explicit key to use for saving and restoring the cache                                                                                                          |
| input      | restore-keys         | An ordered list of keys to check to indicate that a match occurred with the key; a match here is referred to as a cache hit                                         |
| input      | upload-chunk-size    | Chunk size used to split up large files during upload, in bytes                                                                                                    |
| input      | enableCrossOs​Arch⁠ive | Optional boolean—allows Windows runners to save or restore caches that can be restored or saved respectively on other platforms                                   |
| input      | fail-on-cache-miss   | Fail workflow if cache entry is not found                                                                                                                         |
| input      | lookup-only          | See if a cache entry exists for the given input(s) (key, restore-keys) without downloading the cache                                                               |
| output     | cache-hit            | Simple boolean to indicate if an exact match is found for a key                                

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
      
# if anything is changed within module1 then it will be triggered but if the change is also within module1/data then it will not triggered as it has been specified
```

If you’re not familiar with the meaning of the ** symbol, in glob syntax, it matches filenames and directories recursively. Essentially, it matches on anything in a tree structure under the specified path.

If you are triggering off of a push or pull request event, you can refine those more to only trigger off of changes to particular file paths