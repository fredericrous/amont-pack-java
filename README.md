# amont-pack-java

Spotless for a JVM repository, as an [amont](https://github.com/fredericrous/amont)
pack. Two rows, one for Maven and one for Gradle.

```console
$ amont add github:fredericrous/amont-pack-java@v1
github:fredericrous/amont-pack-java @ 8f3c2a1 declares:
    pre-commit  spotless-maven   *.java+pom.xml                        block  mvn -q spotless:check
    pre-commit  spotless-gradle  *.java+build.gradle,build.gradle.kts  block  ./gradlew -q spotlessCheck

amont.conf changed — these commands cannot run until you review them:
    amont trust
```

That second paragraph is not a formality. `amont add` copies text into your
`amont.conf` and stops; the manifest's fingerprint changes, so **every**
declared check — these and any you wrote yourself — is inert until a human
runs `amont trust`. Adding a pack is never the moment anything becomes
runnable.

## Which amont you need

Newer than **1.25.0**. In 1.25.0 and earlier the `+` opt-in was tested against
the files you STAGED rather than the files the repository carries, so these
rows ran only on a commit that also staged `pom.xml` — which is to say, almost
never. Both rows here are gated, so on 1.25.0 this pack is effectively inert.
`amont --version` tells you what you have.

## What you get

Both rows run [Spotless](https://github.com/diffplug/spotless) in check mode
on a commit that stages Java. Neither fixes anything: a pack that rewrites
your working tree is a much larger thing to install than one that complains,
and amont gates that separately behind `amont.fix` anyway.

Whichever build system you don't use stays inert, and says so:

```console
$ amont list --all          # in a Maven repository
  ● spotless-maven (declared)
  ○ spotless-gradle (declared)  inert here — needs .java + build.gradle | build.gradle.kts
```

## Why this is a pack and not a built-in

amont ships ~37 built-in checks, chosen by how many repositories they light up
in. Docker, Helm and shell were worth building in. Java, like Ruby, PHP, .NET
and Swift, was not — which is exactly the case packs exist for. A pack costs
its author a repository and its user one `amont add`, and nobody has to argue
about whether it belongs in the tool.

# Writing your own

A pack is **any git repository with an `amont.pack` at its root**. That is the
entire requirement — no manifest schema, no registry, no build step. This
repository is two files and one of them is the licence.

## The rows

`amont.pack` is written in exactly the syntax of a repository's own
[`amont.conf`](https://fredericrous.github.io/amont/custom-checks.html) —
five whitespace-separated fields, `#` comments, blank lines ignored:

```
# stage       name             scope           severity  command
pre-commit    spotless-maven   *.java+pom.xml  block     mvn -q spotless:check
```

## Gate every row. This is the rule that matters

The `scope` column's `+` splits into *what the change touches* and *what the
repository carries*:

```
*.java + pom.xml
^^^^^^   ^^^^^^^
staged   the repository has one
```

Both sides are comma-separated, each side is an OR, and the two halves are an
AND. `*.java+build.gradle,build.gradle.kts` reads *"a staged `.java`, in a
repository carrying either Gradle build file."*

Your rows land in somebody else's repository. Without a gate, a packaged
linter fires on every matching file whether or not that repository configured
the tool, and simply errors — which is how a useful pack becomes an
uninstalled one. Gate on the thing that makes your command work: the build
file, the tool's config file, the lockfile.

Opt-in markers match a **basename anywhere in the tree**, so a `pom.xml` in a
submodule opts the repository in — which is what you want for a multi-module
build.

## What a pack may not carry

Checks, and nothing else. `tool` version pins, and `severity` / `skip` / `set`
policy lines, are refused — and the **whole pack** is refused with them, not
just the offending row. A half-applied pack is a manifest neither side asked
for.

The reasoning is worth understanding before you go looking for the escape
hatch: a `skip` line could silence the installing repository's secrets scan,
and a `set` could raise its large-file ceiling. Proposing commands somebody
will read is one thing; quietly changing what their existing checks do is
another. Those decisions stay with the repository installing you.

You also cannot take a built-in's id. `pre-commit spotless-maven` is fine;
`pre-commit clippy` is not, because it would shadow `pre-commit-clippy` or
silently lose to it.

## Publishing

Tag it, and tell people to pin the tag:

```sh
git tag v1.0.0 && git push origin v1.0.0
git tag -f v1 && git push -f origin v1     # moving major alias, optional
```

A moving alias is safe here in a way it is not for a CI action. `amont add`
resolves the ref with `git ls-remote` **before** fetching, refuses anything
that isn't the commit it resolved, and writes the **commit id** — never the
tag — into the consumer's manifest, between `# amont:pack:start` /
`# amont:pack:end` markers. So `@v1` picks today's commit at install time and
then stops moving. Re-running `amont add` replaces the block in place.

`<source>` may be `github:owner/repo`, `forgejo:host/owner/repo`, or any git
URL — including a local path, which is all you need to test a pack before you
push it anywhere:

```sh
amont add ../my-pack --dry-run
```

## A limit to design around

The opt-in side is an OR, so you cannot express *"Maven **and** Checkstyle
configured"*. This pack would otherwise carry a `checkstyle-maven` row; gated
on `checkstyle.xml` alone it would misfire in a Gradle repository that has
one, and gated on `pom.xml` alone it would run in Maven repositories that
never configured Checkstyle.

When you hit that, the honest options are to gate on the build file and pick a
command that no-ops when unconfigured, or to ship two narrower packs. Reaching
for the ungated row is the one thing not to do.

## Licence

MIT.
