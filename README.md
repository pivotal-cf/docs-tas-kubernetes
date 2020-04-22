# Documentation for Tanzu Application Service for Kubernetes

Require [this bookbind repo](https://github.com/pivotal-cf/docs-book-pas-kubernetes) to run locally.

Adding Images, guideline: https://docs-wiki.cfapps.io/wiki/image/working-with-images.html

HTML styles: https://docs-wiki.cfapps.io/wiki/style/html-classes.html


## Where is the book repo?
https://github.com/pivotal-cf/docs-book-tas-kubernetes

The _book_ repo uses these branches:

* **Edge** builds from the **master** content branch in this repo.
Pipeline [here](https://concourse.run.pivotal.io/teams/cf-docs/pipelines/pas-kubernetes?group=tas-kubernetes-edge).
* **Master** builds from the published content branches in this repo. Pipeline [here](https://concourse.run.pivotal.io/teams/cf-docs/pipelines/pas-kubernetes?group=pas-kubernetes-0-1).

## Branches in this (content) repo

All documentation for the next unreleased version of Tanzu Application Service for Kubernetes is in `master`.

Always make changes you want carried forward in the master branch. This includes:

* Unreleased features
* Doc bug fixes
* Doc reorganization or enhancement

| Branch Name| Status | Use for version… |
|------------| -------|----------|
| master     | in development | v0.2 (staged here: http://docs-pcf-staging.cfapps.io/tas-kubernetes/0-n/) |
| 0.1        | live, published | v0.1 (http://docs.pivotal.io/tas-kubernetes/0-1/) |
| 2.6.0      | obsolete | v2.6.0-alpha.1 obsolete and removed from docs.pivotal.io and redirect made. Keep branch for historical record. Pipeline already down. (https://docs.pivotal.io/pas-kubernetes/2-6-0-alpha-1 redirects to new docs.) |
| wip.       | obsolete | Okay to delete? Over 10 months old. |

### Cherry picking to and from MASTER

1. Always cherry-pick any changes to live branches into **master** if you want those changes carried forward.

2. If necessary, immediately cherry-pick/copy changes from **master** that you want to push immediately to production into the appropriate live branch above.


## How to Cherry-pick from one Branch to Another
1. Make changes in the first branch (usually `master`), commit them, and then push them to the repo.
2. Copy part of the SHA for the above commit. To find this, you can do a `git log`, or look at the list of commits in the github repo.
3. Checkout the second branch, where you want to copy the changes you made in the first branch.
4. Run this command, using the SHA snippet you copied above:
    `git cherry-pick <SHA_SNIPPET>`<br><br>
    For example: `git cherry-pick 5dc22fe00`

    Do the cherry-pick immediately to lessen the chances of conflicts.
    Otherwise, you may need to resolve conflicts in order to complete the cherry-pick.

5. Do a `git push` after the cherry-pick is complete.<br><br>
