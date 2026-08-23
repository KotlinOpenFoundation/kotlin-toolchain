# Publishing and consuming this fork

[![Published version](https://img.shields.io/maven-metadata/v?metadataUrl=https%3A%2F%2Fkotlinopenfoundation.github.io%2Fkotlin-toolchain%2Fmaven%2Forg%2Fjetbrains%2Fkotlin%2Fkotlin-cli%2Fmaven-metadata.xml&label=published%20version)](https://kotlinopenfoundation.github.io/kotlin-toolchain/)
[![Latest release](https://img.shields.io/github/v/release/KotlinOpenFoundation/kotlin-toolchain?filter=v*&label=distribution)](https://github.com/KotlinOpenFoundation/kotlin-toolchain/releases)
[![Publish](https://img.shields.io/github/actions/workflow/status/KotlinOpenFoundation/kotlin-toolchain/publish-fork-distribution.yml?label=publish)](https://github.com/KotlinOpenFoundation/kotlin-toolchain/actions/workflows/publish-fork-distribution.yml)
[![Rebase](https://img.shields.io/github/actions/workflow/status/KotlinOpenFoundation/kotlin-toolchain/rebase-fixes-onto-upstream.yml?label=rebase%20onto%20upstream)](https://github.com/KotlinOpenFoundation/kotlin-toolchain/actions/workflows/rebase-fixes-onto-upstream.yml)

The version badges read what this fork publishes: the first names the version a project resolves,
the second the release carrying its distribution archive.

This fork carries the fix from
[KTC-5509](https://youtrack.jetbrains.com/issue/KTC-5509): the IDE sync now generates and keeps the
cinterop klibs of `.def` files that a plugin registers through `generated: cinteropDefinitions:`.
Until the fix is merged upstream, projects can use the fork's own distribution.

A build of this fork is named after the upstream development version it is based on, and says that
it is a fork of it: `0.12.0-dev-4300-fork-kof-1` is this fork of `0.12.0-dev-4300`. No version is
pinned in this repository - the version line comes from `next-amper-version.txt` of the sources, the
build number from the newest version upstream has published, and the suffix from the `suffix` input
of the publishing workflow - so rebasing the fixes onto a newer upstream is all it takes for the next
publication to carry a newer name. The landing page names the version and the upstream commit the
build was made from.

## Branches

`main` carries only what publishes this fork: the workflow, this document, and the version the
distribution is published under. Each fix sits on its own branch based on the upstream `main`, so it
stays a clean patch to rebase and to offer upstream:

| branch                                        | what it carries                                                    |
|-----------------------------------------------|--------------------------------------------------------------------|
| `main`                                        | the workflows and this document, and nothing of the sources        |
| `fix/generate-klibs-include-plugin-cinterops` | generate the cinterop klibs of plugin-contributed defs during sync |
| `fix/plugin-api-from-distribution-repository` | take the plugin API from the repository the distribution came from |

There is no branch holding the two together. The publishing workflow merges every `fix/*` branch into
its own working tree before it builds, and pushes none of it, so the assembled state exists only for
the duration of a run and the landing page lists the fixes that went into the build. A fix branch is
therefore expected to hold one commit, and to be based on the upstream `main`.

The publishing workflow runs in three phases - `build`, `archive`, `maven` - so that a failure says
which one broke and a phase can be re-run without building the distribution again. The badges above
show the state of the workflow as a whole, since GitHub reports a status per workflow rather than per
phase.

`Rebase fixes onto upstream` keeps them that way. It runs nightly, and on demand, because a fork is
told nothing when the repository it was forked from moves: it fetches the upstream `main`, rebases
each `fix/*` branch onto it, and pushes the result. A branch that does not rebase cleanly is left
exactly as it was, and an issue is opened naming it. Rebasing changes nothing about what is published
until the distribution is published again.

Both workflows live on `main` because that is where GitHub requires them: a scheduled workflow runs
from the default branch only, and a workflow has to be on the default branch to be dispatchable.
Pushing is confined to the `fix/*` branches for a second reason - the token of a workflow run may not
push changes under `.github/workflows`, so a branch that merged the machinery could not be maintained
by a workflow at all.

## Installing it in a project

The commands below name `0.12.0-dev-4300-fork-kof-1`; the
[landing page](https://kotlinopenfoundation.github.io/kotlin-toolchain/) names the version that is
published now, together with its checksum.

A project that already has Kotlin wrapper scripts - the ones from an upstream release, or an older
build of this fork - switches to this fork with its own `update` command:

```
./kotlin update -r https://kotlinopenfoundation.github.io/kotlin-toolchain/maven --target-version 0.12.0-dev-4300-fork-kof-1
```

That replaces `kotlin` and `kotlin.bat` with the ones published here, after running the new version
once to check that it works. The published wrappers carry the version, the checksum and the address
of this repository, so nothing else has to be configured: no environment variable, and no repository
declared in any manifest.

`--target-version` is not optional. `update --dev` asks the repository for the latest development
version, and the running - upstream - CLI drops any version whose name carries a suffix after the
build number, which every version of this fork does, on purpose, so that a fork build is never
mistaken for an upstream one.

A project without wrapper scripts can take them directly instead:

```bash
base=https://kotlinopenfoundation.github.io/kotlin-toolchain/maven/org/jetbrains/kotlin/kotlin-cli/0.12.0-dev-4300-fork-kof-1
curl -fsSLo kotlin "$base/kotlin-cli-0.12.0-dev-4300-fork-kof-1-wrapper" && curl -fsSLo kotlin.bat "$base/kotlin-cli-0.12.0-dev-4300-fork-kof-1-wrapper.bat" && chmod +x kotlin
```

```powershell
$base = "https://kotlinopenfoundation.github.io/kotlin-toolchain/maven/org/jetbrains/kotlin/kotlin-cli/0.12.0-dev-4300-fork-kof-1"
iwr "$base/kotlin-cli-0.12.0-dev-4300-fork-kof-1-wrapper" -OutFile kotlin; iwr "$base/kotlin-cli-0.12.0-dev-4300-fork-kof-1-wrapper.bat" -OutFile kotlin.bat
```

Either way, `./kotlin --version` then reports `0.12.0-dev-4300-fork-kof-1`, downloading the distribution on first use and
verifying its checksum.

To install the toolchain for the user rather than for one project, run the installer published beside
the distribution; it points at this repository as well:

```bash
curl -fsSL https://kotlinopenfoundation.github.io/kotlin-toolchain/maven/org/jetbrains/kotlin/kotlin-cli/0.12.0-dev-4300-fork-kof-1/kotlin-cli-0.12.0-dev-4300-fork-kof-1-installer.sh | sh
```

A project that wants to keep its own wrapper scripts can point them here by hand instead: set
`kotlin_cli_version` and `kotlin_cli_sha256` to the values on the
[landing page](https://kotlinopenfoundation.github.io/kotlin-toolchain/), and either set
`KOTLIN_CLI_DOWNLOAD_ROOT` in the environment or replace the default repository URL inside the
scripts.

Modules of type `jvm/amper-plugin` need nothing extra. The distribution resolves its own
`amper-extensibility-api` from the repository it was downloaded from, so the artifact of a fork
version is found without the project repeating the URL.
## Publishing

The `Publish fork distribution` workflow builds the distribution and the libraries and publishes
them as a static Maven repository on GitHub Pages:

The JRE, the distribution that builds this one, and the resolved dependencies are cached between
runs under `~/.cache/JetBrains/Kotlin`, keyed by the wrapper and the version files. The project
build directory is deliberately not cached: incremental state that went wrong would end up
published and pinned by checksum.

- Run it manually from the Actions tab, optionally overriding the version, or
- push a tag matching `dist-v*`, which additionally attaches the distribution archive to a release.

The repository must have Pages enabled with **Deploy from a branch** as the source, serving
`maven-site` at its root (Settings → Pages → Build and deployment). The workflow writes that
branch; GitHub serves it. Nothing large is committed to it, because the distribution archive goes
to a release instead, well clear of the 100 MB per-file limit of Git.

The workflow prints the SHA-256 of the distribution archive and repeats it on the published landing
page, because the wrapper of a consuming project verifies it. The archive is not byte-reproducible,
so publishing the same version twice changes that checksum, and a project pinned to the old one
fails to download the distribution once its toolchain cache is cleared. The workflow warns when it
happens; prefer bumping the fork suffix over republishing a version that something already uses.

### Where each thing is served from

The distribution archive is around 214 MB, and a Pages site may hold 1 GB in total and serve about
100 GB a month, so keeping every archive on the site would run out of room after a handful of
versions and out of traffic sooner than that. The archive is therefore attached to a release of its
own, tagged `v<version>`, where an asset may be 2 GB and there is no practical limit on how many
versions are kept.

Everything else - poms, libraries, wrapper scripts, installers, checksums, `maven-metadata.xml` - is
kilobytes, and is served from the Pages site, which keeps every version that was ever published. The
wrapper scripts of a version name the release asset of that same version, so a consumer needs to know
about one address only, and the scripts of versions published earlier are left untouched by later
runs.

A Pages deployment replaces the whole site, and the local Maven repository of a fresh runner holds
only the version that was just built, so the workflow carries the accumulated tree between runs as a
`maven-site` branch, which holds exactly what the site served last time: the run restores it, merges the new version
into it, and force-pushes the result back as a single commit. The branch is generated content, not
source: it carries the site and nothing else, which is also why a workflow may write it, since the
token of a run may not push changes under `.github/workflows`. Some such store is unavoidable,
because a Pages deployment has no memory of what the previous one served. Nothing is pruned
any more; the run reports the size of the site, which grows by tens of kilobytes per version.

What gets published:

| coordinates                                                                      | why a consumer needs it                                                                                        |
|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| `org.jetbrains.kotlin:kotlin-cli:<version>` (`wrapper`, `installer` classifiers) | the scripts a project installs; the `dist` archive itself is a release asset                                   |
| `org.jetbrains.amper:amper-extensibility-api:<version>`                          | any module with `product: jvm/amper-plugin` implicitly depends on it, at the version of the CLI that builds it |
| the other `org.jetbrains.amper:*` libraries                                      | dependencies of the above, and of the Gradle and IDE integrations                                              |

## Why GitHub Pages

The wrapper downloads the distribution with a plain, unauthenticated HTTPS GET from
`$KOTLIN_CLI_DOWNLOAD_ROOT/org/jetbrains/kotlin/kotlin-cli/<version>/kotlin-cli-<version>-dist.tgz`,
and verifies its SHA-256. That constrains the hosting:

- **GitHub Pages** serves any path over anonymous HTTPS, which fits the layout the wrapper expects.
  A site is limited to 1 GB and to a soft 100 GB of traffic per month. The distribution archive is
  around 214 MB, so about four versions fit, which is what the `keep` input of the workflow bounds -
  see [Retention](#retention). This is what the workflow uses.
- **GitHub Releases** allow assets up to 2 GB and need no credentials, but an asset URL cannot
  contain the directory structure of a Maven repository, so the wrapper cannot download from it. The
  workflow attaches the archive to a release anyway, for people who install the distribution by hand
  or seed the cache directory directly.
- **GitHub Packages** requires a token even for reading a public package, so the wrapper fails
  against it. It remains usable for the libraries alone, if every consuming project is configured
  with credentials.
- **A third-party Maven host** (Cloudsmith, JFrog, an object store behind a CDN) works as well, and
  is worth it only if the traffic outgrows Pages.

## Keeping the fork in sync

`Rebase fixes onto upstream` does this nightly, so in the ordinary case there is nothing to do but
publish again: the new build is named after the newer upstream version, which is what tells consumers
the builds apart. Raise the number in the `suffix` input when publishing twice against the same
upstream version, so that those two are distinguishable as well.

When a fix stops rebasing cleanly, the workflow leaves the branch alone and opens an issue. Rebase it
by hand, push it, and the nightly run takes over again.
