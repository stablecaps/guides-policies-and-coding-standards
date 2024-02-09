# Release strategy (implementation) 2

## Overview

Unfortunately many of these rules can only be applied at the repo level as a Github enterorise subscription is required to do it at the org level. Thus, it is of interest to make cookie cutter projects with all these setting enabled.


## Squash merge
1. Feature branches should be cut from main
2. The PR should be merged as a squash commit.


## Github settings
1. Disallow merge commits & squash commits (repo settings)
2. Set squash merge to take PR title & description
2. protect master and release branches
    - Require PR with approval


## Branching strategy

Use Github Flow with feature-branching strategy and squash commits.

<img src="images/git-flow.png" alt="GitHub Flow" width="330" height="522"/>

#### GitHub Flow Considerations
While working with the GitHub flow branching strategy, there are six principles you should adhere to to ensure you maintain good code.

1. Any code in the main branch should be deployable.
2. Create new descriptively-named branches off the main branch for new work, such as feature/add-new-payment-types.
3. Commit new work to your local branches and regularly push work to the remote.
4. To request feedback or help, or when you think your work is ready to merge into the main branch, open a pull request.
5. After your work or feature has been reviewed and approved, it can be merged into the main branch.
6. Once your work has been merged into the main branch, it should be deployed immediately.

#### The Benefits of GitHub Flow
1. Of the three Git branch strategies we cover in this post, GitHub flow is the most simple.
2. Because of the simplicity of the workflow, this Git branching strategy allows for Continuous Delivery and Continuous Integration.
3. This Git branch strategy works great for small teams and web applications.

#### The Challenges of GitHub Flow
1. This Git branch strategy is unable to support multiple versions of code in production at the same time.
2. The lack of dedicated development branches makes GitHub flow more susceptible to bugs in production.

## Links
1. [PSR Discussion](https://github.com/python-semantic-release/python-semantic-release/issues/816)
2. [githubflow.github.io](https://githubflow.github.io/)
3. [git-branch-strategy](https://www.gitkraken.com/learn/git/best-practices/git-branch-strategy)


## PSR & Tagging

See [PSR Discussion](https://github.com/python-semantic-release/python-semantic-release/issues/816)

**Image from above discussion:**
<br>

<img src="images/tagging_example.png" alt="Tagging example" width="793" height="489"/>

> I built this example workflow to illustrate an example workflow I would do to include supporting of 2 concurrent major versions (ie, v2.0.0 latest, & v1.0.0 maintenance). I built it using my version of GitHub Flow but after the squash merges it basically ends up looking like trunk-based development.
> If I had concurrent development efforts of multiple teammates, there would be more branches waiting for completion and merge into main. This should not be a slow process, for merge into main. IMO, feature branches should focus on one aspect of the code and should not live very long to prevent complicated rebases and merge conflicts.
>
> You will see that I branch from the v1 tag and then make the change. Since the maintenance patch has to be manual, the creation of the branch is manual and then I can call PSR on that branch. It's a personal thing not to keep branches around as they can always be re-created; I would next delete v1.x.x as it doesn't serve any purpose.
>
>If you use a Visual Git program such as the Git Graph VSCode extension, branches create clutter so I remove them as much as possible. I left some of the branches at the bottom to illustrate the squash merges but ultimately these would be pruned after merge.
>