# Neo4j-Changelog

A tool to generate changelogs based on GitHub pull requests. As far as
this tool is concerned, every entry in the changelog corresponds to
one PR on GitHub.

The major feature of `neo4j-changelog` is the fact that it actually
uses git to figure out which version a PR belongs to. Thus there is
never any need to "figure out" which version a change was first
introduced in, the tool will do all that for you.

As a result it becomes easy to generate changelog for multiple
versions, even if you are running parallel branches where some PRs are
effectively merged into several branches.

## Cloning

This repo uses a submodule so the best way is to do

```
git clone --recursive https://github.com/spacecowboy/neo4j-changelog.git
```

or if you read this after your first build failed (due to a missing submodule), do this

```
git clone https://github.com/spacecowboy/neo4j-changelog.git
cd neo4j-changelog
git submodule update --init --recursive
```

## How to build

Use `./gradlew tasks` to list possible tasks. But you probably want either

*  `installDist`
   which will build a runnable script for you at `build/install/neo4j-changelog`

* `distTar` or `distZip`
   which builds a runnable script and packages it up for you under `build/distributions`

You can then just run the executable under `build/install/neo4j-changelog/bin/neo4j-changelog`.

## How to use

Output of `neo4j-changelog --help`:

```
usage: neo4j-changelog [-h] [-c CONFIG]

Generate changelog for the given project.

optional arguments:
  -h, --help             show this help message and exit
  -c CONFIG, --config CONFIG
                         Path to config file (default: changelog.toml)
```

Please
see
[sample-changelog.toml](https://github.com/spacecowboy/neo4j-changelog/blob/master/sample-changelog.toml)
for a complete sample config file.

## How to label PRs

To avoid terminology confusion, *label* refers to GitHub
Issue/PullRequest labels, the ones you assign to PRs through the
GitHub GUI. *tag* refers to a piece of text inside the brackets of a
PR description. *release* refers to what is specified as releases by
GitHub, equivalent to *git tags*.

Only PRs with the label(s) configured in `requiredlabels` are
downloaded from github. From these, a merge-commit is extracted to
determine where (if at all) the PR belongs in the change log. So as to
not consider ALL PRs, it is recommended to label those PRs (with a
GitHub Label) that should be mentioned in the change log with for
example "changelog".

Additional meta-data is also parsed from the PR's description.

### Extra meta-data format

It is possible to override the metadata associated with a PR by adding
some text to the message body of the PR, such as

    changelog: [3.4, packaging] This is a better message

which override the PR's GitHub labels, and the PR title. Versions
inside the brackets can be used to limit the inclusion of a change in
case it was null-forward-merged.

Each PR is sorted under the earliest (by semantic version) release
from which the PR's head commit is reachable. E.g., each PR is placed
under first release which occurs after the relevant merge commit. If
no such release exists, it is placed under `nextheader`.

An exception is if any versions were specified with the changelog tag
in the PR message. If so, that version is compared to `versionprefix`
from the config.

Each PR is sorted under a category (`categories` in config). The first
GitHub label, or tag if present, to match one of the categories will
be used. If no match is found, it is placed under `Misc`.

#### Meta-data examples
Some examples are the best way to illustrate.

A change should be placed under `packaging`, but the change is only
present in 3.3 and 3.4 (e.g. it was null forward merged
to 3.5) and we want the changelog to state "Some fixes to Load CSV"
instead of whatever the PR title is:

```
changelog: [3.3, 3.4, packaging] Some fixes to Load CSV
```

The colon after `changelog` is optional, and it is also OK to write
`cl` instead, so the following is perfectly equivalent:

```
cl [3.3, 3.4, packaging] Some fixes to Load CSV
```

In fact, we could also have placed it across several lines such as

```
changelog
[3.3, 3.4, packaging]
Some fixes to Load CSV
```

New lines within the changelog message should however be avoided.
This will cause an empty line between `to` and `Load` in the generated changelog:

```
cl [3.3, 3.4, packaging] Some fixes to 
Load CSV
```

Each individual piece is of course optional, so if we are fine with
the title of PR being listed in the changelog we could just write:

```
CHANGELOG:[3.3,3.4,packaging]
```

The spacing and case don't matter. Similarly, if the PR is already tagged
correctly and it was not null-merged anywhere, but we are not OK with
the title, this would be fine:

```
cl: Some fixes to Load CSV
```

A separate PR for the same fix was raised for each of version 3.3, 3.4 and 3.5. 
To avoid duplicate entries in the changelogs of 3.4 and 3.5, they should
have these changelog messages, respectively:

```
cl: [3.3] Some fixes to Load CSV
```

```
cl: [3.4] Some fixes to Load CSV
```

```
cl: [3.5] Some fixes to Load CSV
```

A PR solves a Github issue, which should be linked to from the changelog:

```
cl: Fixes an [issue](https://github.com/neo4j/neo4j/issues/12345) with Load CSV
```

In case you were wondering, writing nothing will be ignored. These are
all equivalent, and they are effectively ignored when parsed:

```
changelog:
cl:
CHANGELOG
Cl
ChAngELog []
cL[]
```
