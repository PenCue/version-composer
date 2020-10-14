# mrversion 

##### based on Version mrversion v1.2.0.1

Creates a new version identifier to use with your builds and releases.
#### Rename and major changess
`mrversion` based on [version_mrversion](https://github.com/juliansangillo/version-mrversion) by [Julian Sangillo](https://github.com/juliansangillo)
The major changes are 

* inclusion of github PR support using branch names, 
* removal of build numbering, 
* some refactoring (mainly around logging), 
* removal of options
* addition of more exposed variables (including the prefix "v" )
## Features
* Return as output a version with and without a release tag appended at the end
* Control how your versions are formatted
* Choose what branches you want new versions to be based off of at each stage of development
* Choose what release tags to use at each stage of development
* increase version numbers based on the name of the feature branch at PR clo
## Version Format
\<prefix\>\<major\>.\<minor\>.\<patch\>.\[.\<release-tag\>\]

\<prefix\>\<major\>.\<minor\>\[.\<release-tag\>\]

<prefix\>\<major\>\[.\<release-tag\>\]

### Examples
* `v2`

* `v3.2`

* `v2.3.42.rc`

* `v3.2.15.unstable`

* `v2.3.42`

  
# Usage
See [action.yml](https://github.com/PenCue/mrversion/blob/master/action.yml)
## How it knows your latest version?
In order to create the next version, it needs to know what the latest version is. It does this by getting the latest annotated tag in your repo. It then strips off the 'v' character that is prepended to the tag. This string should be numbers separated by periods.
## Branching Strategy
Version mrversion assumes that there are three branches in your repo with each branch representing a stage of development. There should be a development branch, an integration branch, and a stable branch. The development branch is the most unstable branch and is used to release new work that has yet to be tested. The integration branch is for features ready to be integration tested. The stable branch is for final production releases. These could be customized to be any branch. The defaults for development, integration, and stable are `dev`,`int`, and `main` branches respectively.
### Github flow
only changes to development branch via Pull requests -> bump version. Integration and main branches follow development versions.  

push: new version -> `dev`  
PR: `dev` -> `int`  
PR: `int` -> `main`

## Version Bumping
There are 3 different revision types that determine how a version is incremented when a new one is created in the development branch. They are: major, minor and patch. The revision type during execution is based on the commit messages being included since the latest version or the name of the branch of the PR branch It is assumed your commit messages are following the [Angular Commit Message Conventions](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines) with the addition of 'build' and 'patch' types. Higher priority changes always take precedence. Major changes are always more important than minor ones. Likewise, minor is higher than build and patch, while build and patch are about the same. If none of the commit messages below exist, then it will fall back to the preferred-type. The preferred-type can be either build or patch and changing it will change whether you prefer to use the build value or the patch value for most types of changes. This can be overwridden during execution by specifying build or patch in one of your commits. By default preferred-type is build.
| Revision<br>Type | Commit Message | Branch name |
|-|-|-|
| Patch <br>Revision | `patch: stop graphite breaking when too much pressure applied`<br>`patch(pencil): stop graphite breaking when too much pressure applied` | `fix/3-graphite-breakage` |
| Minor<br>Revision | `feat: add 'graphiteWidth' option`<br>`feat(pencil): add 'graphiteWidth' option`<br><br>`perf: remove graphiteWidth option`<br>`perf(pencil): remove graphiteWidth option` | `feat/4-removeing-graphite-width-option` |
| Major<br>Revision | `BREAKING CHANGE: The graphiteWidth option has been removed.<br>The default graphite width of 10mm is always used for performance reasons.` | `breaking/8-no-more-width-options` |
## Release Tags
The release tag is a string of capitalized ASCII characters marking the version as a type of release based on branch (example: 1.0.0.`int`). Using release tags are advisable since multiple branches are being used here. It is possible to customize what tags are used for which branches. By default, the dev branch uses the `unstable` tag, the `int` branch uses the '`rc`' tag, and the stable branch uses no tag. You can enter an empty string for no tag. It is recommended though that all branches have different tags, otherwise there will be conflicting versions.
### What about versioning on `int` and stable branches?
When you merge a pull request (or push into `int` or production branches), the version is not incremented as you are not introducing a new instance of your project. Instead, you are promoting an already existing version to a higher branch. The only thing that changes in this case is the release tag. Below examples show the flow when the latest version is 2.1.0 and a new build is pushed.
### local -> dev
`2.1.1.unstable`

### dev -> `int`
`2.1.1.rc`
### `int` -> master
`2.1.1`
## Inputs
### stable-branch
Name of branch in your repo that will be used for stable releases. Default is `main`
### int-branch
Name of branch in your repo that will be used for `int` releases. Default is `int`
### dev-branch
Name of branch in your repo that will be used for development releases. Default is `dev`
### stable-tag
Custom release tag name for stable releases. By default, stable releases have no stable tag.
### int-tag
Custom release tag name for `int` releases. Default is `rc`
### dev-tag
Custom release tag name for development releases. Default is `unstable`
### version-prefix
Prefix for version number. Default is `v`
## Ouputs
### full-version
The new version with the release tag appended at the end.
### major-version
The new major version only (v)
### major-minor-version
The new major.minor
### version
Version without the release tag at the end.
### tag

release tag

#### dot-tag

release tag with dot prefix if not empty

#### version-prefix

prefix for the version number 

### is-prerelease

True if this version is an unstable release (dev or `int` branch) or false otherwise.

## Setup
The workflows is triggered on a PR close and when the PR is closed AND merged. It needs the `fetch-deep 0` to make sure it as enough history. (this may change ). It will tag the checkout version and push it. 

### Workflow 

```yml
name: pr-version-bump-ci

on:
  pull_request:
    types: [closed]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Mr Version
        if: github.event_name == 'pull_request' && github.event.action == 'closed' && github.event.pull_request.merged == true
        id: mrversion
        uses: PenCue/mrversion@v2
        # all the defaults
        with:
          stable-branch: master
          int-branch: int
          dev-branch: dev
          stable-tag: null
          int-tag: rc
          dev-tag: unstable
          version-prefix: v

      # Runs a set of commands using the runners shell
      - name: Set Tag
        if: github.event_name == 'pull_request' && github.event.action == 'closed' && github.event.pull_request.merged == true
        run: |
          echo Release ${{ steps.mrversion.outputs.full-version }}
          git tag ${{ steps.mrversion.outputs.full-version }} $GIT_SHA
          git push origin --tags ${{ steps.mrversion.outputs.full-version }} 

```
## See Also

