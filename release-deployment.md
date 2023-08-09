| [Home](README.md) ▸ **Release Deployment** |
|-----|

# Release Deployment

Use this strategy for projects where features get bundled into a release and then
deployed all at once.

We follow the [**Gitflow**](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
workflow as closely as possible. This page showcases common development scenarios
and how to deal with them from a branching point of view.

- [Branches Overview](#branches-overview)
- [Use Cases](#use-cases)
  - [Develop a new feature](#develop-a-new-feature)
  - [Develop multiple features in parallel](#develop-multiple-features-in-parallel)
  - [Create and deploy a release](#create-and-deploy-a-release)
  - [Change in plan, pull a feature from a release](#change-in-plan-pull-a-feature-from-a-release)
  - [Change request](#change-request)
  - [Production hot fix](#production-hot-fix)
- [Migrate a legacy project](#migrate-a-legacy-project)

## Branches Overview

![Release Deployment workflow](images/release-deployment.png)

| Branch        | Protected? | Base Branch | Description    |
| :-------------|:-----------|:------------|:---------------|
| `master`      | YES        | N/A         | What is live in production (**stable**).<br/>A pull request is required to merge code into `master`. |
| `develop`     | YES        | `master`    | The latest state of development (**unstable**). |
| feature       | NO         | `develop`   | Cutting-edge features (**unstable**). These branches are used for any maintenance features / active development. |
| `release-vX.Y.Z` | NO      | `develop`    | A temporary release branch that follows the [semver](http://semver.org/) versioning. This is what is sent to UAT.<br/>A pull request is required to merge code into any `release-vX.Y.Z` branch. |
| bugfix        | NO         | `release-vX.Y.Z` | Any fixes against a release branch should be made in a bug-fix branch. The bug-fix branch should be merged into the release branch and also into develop. This is one area where we’re deviating from GitFlow. |
| `hotfix-*`    | NO         | `master`    | These are bug fixes against production.<br/>This is used because develop might have moved on from the last published state.<br/>Remember to merge this back into develop and any release branches. |

## Use Cases

### Develop a new feature

![Feature branch](images/feature-branch.png)

1. Make sure your `develop` branch is up-to-date

1. Create a feature branch based off of `develop`

   ```
   $ git checkout develop
   $ git checkout -b MYTEAM-123-new-documentation
   $ git push --set-upstream MYTEAM-123-new-documentation
   ```

1. Develop the code for the new feature and commit. Push your changes often. This
   allows others to see your changes and suggest improvements/changes as well as
   provides a safety net should your hard drive crash.

    ```
    $ ... make changes
    $ git add -A .
    $ git commit -m "Add new documentation files"
    $ ... make more changes
    $ git add -A .
    $ git commit -m "Fix some spelling errors"
    $ git push
    ```

1. Navigate to the project on [Github](www.github.com) and open a pull request with
   the following branch settings:
   * Base: `develop`
   * Compare: `MYTEAM-123-new-documentation`

1. When the pull request was reviewed, merge and close it and delete the
   `MYTEAM-123-new-documentation` branch.

### Develop multiple features in parallel

There's nothing special about that. Each developer follows the above [Develop a new feature](#develop-a-new-feature) process.

### Create and deploy a release

![Create and deploy a release](images/release-release.png)

1. Merge `master` into `develop` to ensure the new release will contain the
   latest production code. This reduces the chance of a merge conflict during
   the release.

   ```
   $ git checkout develop
   $ git merge master
   ```

1. Create a new `release-vX.Y.Z` release branch off of `develop`.

   ```
   $ git checkout -b release-vX.Y.Z
   $ git push --set-upstream release-vX.Y.Z
   ```

1. Stabilize the release by using bugfix branches off of the `release-vX.Y.Z` branch
   (the same way you would do a feature branch off of `develop`).

   ```
   $ git checkout release-vX.Y.Z
   $ git checkout -b fix-label-alignment
   $ git push --set-upstream fix-label-alignment
   ... do work
   $ git commit -m "Adjust label to align with button"
   $ git push
   ```

1. When the code is ready to release, navigate to the project on
   [Github](www.github.com) and open a pull request with the following branch
   settings:
   * Base: `master`
   * Compare: `release-vX.Y.Z`
   Paste the Release Checklist into the PR body. Each project should define a release
   checklist. It will vary across projects, but you can refer to the Astro [Release](https://github.com/mobify/astro/blob/develop/RELEASE.md) document
   for an example.

1. At some point in the checklist you will merge the release branch into `master`.
   You can do this by using the "Merge pull request" button on the release PR.

1. Now you are ready to create the actual release. Navigate to the project page
   on Github and draft a new release with the following settings:
   * Tag version: `vX.Y.Z`
   * Target: `master`
   * Release title: `Release vX.Y.Z`
   * Description: Include a high-level list of things changed in this release.
   Click `Publish release`.

1. Merge the `release-vX.Y.Z` into `develop`.

    ```
    $ git checkout develop
    $ git merge release-vX.Y.Z
    $ git push
    ```

1. Finish off the tasks in the release checklist. Once everything is done, close
   the release PR.

1. Delete the release branch on Github.

### Change in plan, pull a feature from a release

**TBD: Discuss**
Mike N: That probably means recreating the release branch, unless we have short-lived release branches

### Supporting old releases

In a release based project old releases can often need some maintenance. Critical bug fixes and strategic feature requests need to be supported on old releases. Old releases often become incompatible with the most recent versions of projects.

In order to support old release versions gitflow has introduced the concept of support branches. Support branches are long living branches created to support major or minor versions of the project. Support branches do not get merged back into `master` or `develop` (this would cause major merge issues which are time consuming and error prone if attempted). Instead commits can be cherry picked from the support branch back into `develop`. 

Support branches can be thought of as the `master` branch for old releases. Support branches for major releases should be named as `support-v<major>.x`. Support branches for minor releases should be named as `support-v<major>.<minor>.x`.

Here is an example of creating a support branch for v1.0 assuming v2.0 of the project has already been released.

1. Create the support branch and release branch for the patch release.

    ```
    // Checkout the tag for the 1.0.0 release
    git checkout v1.0.0

    // Create the long living support branch
    git checkout -b support-v1.x

    // Create the release branch
    git checkout -b release-v1.0.1
    ```

    Note: For subsequent releases (ex v1.0.2) the release branch will be branched off the `HEAD` of `support-v1.x`

1. Make changes in the `release-v1.0.1`. Multiple PRs can be merged into this branch if several changes are necessary.

1. As PRs are merged into the `release-v1.0.1` branch create associated PRs that cherry pick the changes back into `develop`. Ensure that these changes are desired by the team going forward and that they are compatible with the current state of the `develop` branch.

1. Create release PR to merge `release-v1.0.1` into `support-v1.x`.

1. Follow the standard release process treating `support-v1.x` as the `master` branch. As per the standard release process `release-v1.0.1` will get deleted and `support-v1.x` will remain in repo indefinitely.

1. Mark `support-v1.x` as a protected branch in github so that it does not get accidentally deleted.

*Pro-tip*: Try to maintain as few support branches as possible. These branches are expensive to maintain since you will need to cherry pick applicable bug fixes into each support branch seperately.


### Production hot fix

A production hotfix is very similar to a full-scale release except that you do
your work in a branch taken directly off of `master`. Hotfixes are useful in cases
where you want to patch a bug in a released version, but `develop` has unreleased
code in it already.

**TBD: Insert diagram**

1. Create a hot fix branch based off of `master`.

   ```
   $ git checkout master
   $ git checkout -b hotfix-documentation-broken-links
   $ git push --set-upstream origin hotfix-documentation-broken-links
   ```

1. Add a test case to validate the bug, fix the bug, and commit.

   ```
   ... add test, fix bug, verify
   $ git add -A .
   $ git commit -m "Fix broken links"
   $ git push
   ```

1. Navigate to the project on [Github](www.github.com) and open a pull request
   with the following branch settings:
   * Base: `master`
   * Compare: `hotfix-documentation-broken-links`
   Paste your release checklist into the PR and work through the PR to get the
   hotfix into production.

1. At some point in the checklist you will merge the hotfix branch into `master`.
  You can do this by using the "Merge pull request" button on the release PR.

1. Now that the hotfix code is in `master` you are ready to create the actual
   release. Navigate to the project page on Github and draft a new release with
   the following settings:
   * Tag version: `vX.Y.Z`
   * Target: `master`
   * Release title: `Release vX.Y.Z (hotfix)`
   * Description: Include a high-level list of things changed in this release.

  Click `Publish release`.

  *Note: Hotfix releases _are_ actual releases. You should bump at least the
  patch part of the version when releasing a hotfix and so even hotfixes go
  through the process of creating a release like this.*

1. Merge the `hotfix-documentation-broken-links` into `develop`.

   ```
   $ git checkout develop
   $ git merge hotfix-documentation-broken-links
   $ git push
   ```

1. Finish off the tasks in the release checklist. Once everything is done, close
   the hotfix PR.
