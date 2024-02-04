
# Release strategy (research) 1

## Conventional commits
[CC specification](https://www.conventionalcommits.org/en/v1.0.0/)

The Conventional Commits specification is a lightweight convention on top of commit messages. It provides an easy set of rules for creating an explicit commit history; which makes it easier to write automated tools on top of. This convention dovetails with [SemVer](https://semver.org/), by describing the features, fixes, and breaking changes made in commit messages.

The commit message should be structured as follows:
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Explicit examples:
```
feat(logging): added logs for failed signups
fix(homepage): fixed image gallery
test(homepage): updated tests
docs(readme): added new logging table information
```
The commit contains the following structural elements, to communicate intent to the consumers of your library:

1. fix: a commit of the type fix patches a bug in your codebase (this correlates with PATCH in Semantic Versioning).
2. feat: a commit of the type feat introduces a new feature to the codebase (this correlates with MINOR in Semantic Versioning).
3. BREAKING CHANGE: a commit that has a footer BREAKING CHANGE:, or appends a ! after the type/scope, introduces a breaking API change (correlating with MAJOR in Semantic Versioning). A BREAKING CHANGE can be part of commits of any type.
4. types other than fix: and feat: are allowed, for example @commitlint/config-conventional (based on the Angular convention) recommends:
- feat: A new feature
- fix: A bug fix
- docs: Documentation only changes
- style: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
- refactor: A code change that neither fixes a bug nor adds a feature
- perf: A code change that improves performance
- test: Adding missing or correcting existing tests
- chore: Changes to the build process or auxiliary tools and libraries such as documentation generation.
5. footers other than BREAKING CHANGE: <description> may be provided and follow a convention similar to git trailer format.

Additional types are not mandated by the Conventional Commits specification, and have no implicit effect in Semantic Versioning (unless they include a BREAKING CHANGE). A scope may be provided to a commit’s type, to provide additional contextual information and is contained within parenthesis, e.g., feat(parser): add ability to parse arrays.



## Semantic versioning reminder
[SemVer](https://semver.org/)
Given a version number MAJOR.MINOR.PATCH, increment the:

1. MAJOR version when you make incompatible API changes
2. MINOR version when you add functionality in a backward compatible manner
3. PATCH version when you make backward compatible bug fixes
4. Additional labels for pre-release and build metadata are available as extensions to the MAJOR.MINOR.PATCH format.

## Breaking changes
We need an automatic method to detect breaking changes as relying on developers making the correct commits is too fragile.
[From - Breaking up the monolith: Breaking changes](https://dev.to/authress/breaking-up-the-monolith-breaking-changes-14dd)
Some examples of breaking changes might be:

- API interface property type is changed (from int to string for instance)
- Size of the data property is changed, (from 3 to 4 characters)
- Returning an additional enum value in an enum property where you didn’t first explain to the clients that this list can be expanded. The general recommendation is that this isn’t a breaking change. But remember it doesn’t matter if you think it is, it matters if your clients do.
- If you return an inconsistent or different error code or response status code. Returning a 400 instead of a 404 can be considered a breaking change. 404 means something, it’s possible that the 404 was a bug, and the resource really existed. So sometimes making a breaking change is a good thing.
- Allowing the schema type of a property be different in difference circumstances, i.e. returning an int or a string, just don’t do this. While it is possible to document the union types, it's a huge headache for development teams to deal with.
- Requiring a previously optional property or requiring a new header to continue having previous functionality. Clients not sending the header or the property will now get a 400 or 422 back on their response, instead of the previous 2xx.

A potentional method to figure out if a breaking change has been made is the idea to detect unmarked breaking changes by running the previous version's test suite on a new version. If the new version fails the old version's tests, it's probably a breaking change. [See Stephan Bönnemann talk](https://www.youtube.com/watch?v=tc2UgG5L7WM)

## Branching strategies
1. [Git Branching strategies comparison
](https://www.opsatscale.com/articles/Git-branching-strategies-comparison/)
2. [Git Branching Strategies: What Are Different Git Branching Strategies?](https://phoenixnap.com/kb/git-branching-strategy)
3. [Comparing Git branching strategies](https://dev.to/arbitrarybytes/comparing-git-branching-strategies-dl4)
4. [Enhanced Git Flow Explained](https://www.toptal.com/gitflow/enhanced-git-flow-explained)
5. **[How to get rid of develop branch for simplified Git flow](https://devops.stackexchange.com/a/741)**
6. [Release flow 1](https://stackoverflow.com/a/50257357)
7. [Releaseflow](http://releaseflow.org/)
8. [Branching – Release Flow](https://www.linkedin.com/pulse/branching-release-flow-deixei)
9. [Release Flow: How We Do Branching on the VSTS Team](https://devblogs.microsoft.com/devops/release-flow-how-we-do-branching-on-the-vsts-team/)
10. [Power of Branching Strategies and Conventions]
11. [Branching & Merging Strategies – Release Flow](https://dev.to/jeastham1993/branching-merging-strategies-release-flow-18f3)

## Links
1. [Automating Versioning and Releases Using Semantic Release](https://medium.com/agoda-engineering/automating-versioning-and-releases-using-semantic-release-6ed355ede742)
2. [Breaking up the monolith: Breaking changes](https://dev.to/authress/breaking-up-the-monolith-breaking-changes-14dd)
3. [Semantic release with Python, Poetry & GitHub Actions](https://mestrak.com/blog/semantic-release-with-python-poetry-github-actions-20nn)
