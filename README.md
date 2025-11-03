# Coverity Scan Action

**This is not an official Coverity or Synopsys project.**

Make it easy to build your project using
[Coverity Scan](https://scan.coverity.com/)'s tools, and then upload the results
to their site for analysis.  This is great for OSS projects.

# Example

```yaml
# Your .github/workflows/coverity.yml file.
name: Coverity Scan

# We only want to test official release code, not every pull request.
on:
  push:
    branches: [main]

jobs:
  coverity:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: vapier/coverity-scan-action@v1
      with:
        email: ${{ secrets.COVERITY_SCAN_EMAIL }}
        token: ${{ secrets.COVERITY_SCAN_TOKEN }}
```

Make sure to define `COVERITY_SCAN_TOKEN` in your
[project's secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets).

## Live examples

Here's a non-exhaustive list of projects using this action.

* [Breakpad](https://github.com/google/breakpad/blob/HEAD/.github/workflows/coverity.yml) (configure+make)
* [libexpat](https://github.com/libexpat/libexpat/blob/HEAD/.github/workflows/coverity-scan.yml) (cmake+make)
* [LibGD](https://github.com/libgd/libgd/blob/HEAD/.github/workflows/coverity.yml) (cmake+make)
* [Minijail](https://github.com/google/minijail/blob/HEAD/.github/workflows/coverity.yml) (make)
* [ncompress](https://github.com/vapier/ncompress/blob/HEAD/.github/workflows/coverity.yml) (make)
* [OpenRC](https://github.com/OpenRC/openrc/blob/HEAD/.github/workflows/coverity.yml) (meson+ninja)
* [pax-utils](https://github.com/gentoo/pax-utils/blob/HEAD/.github/workflows/coverity.yml) (make)
* [app-mesh](https://github.com/laoshanxi/app-mesh/blob/HEAD/.github/workflows/code-scan-coverity.yml) (cmake+make)

# Usage

```yaml
- uses: vapier/coverity-scan-action@v1
  with:
    # Project name in Coverity Scan.
    #
    # This should be as it appears on the Coverity Scan website.
    # Find it in your dashboard:
    # https://scan.coverity.com/dashboard
    #
    # For example, a GitHub project will look like "gentoo/pax-utils".
    #
    # NB: This value is case-sensitive and must match what your GitHub project
    # is registered as exactly!
    #
    # Default: ${{ github.repository }}
    project: ''

    # Secret project token for accessing this project in Coverity Scan.
    #
    # Find this in the project's "Project Settings" tab under "Project token" on
    # the Coverity Scan website.
    #
    # This value should not be specified in the yaml file directly.  Instead it
    # should be set in your repositories secrets.  "COVERITY_SCAN_TOKEN" is a
    # common name here.
    # https://docs.github.com/en/actions/security-guides/encrypted-secrets
    #
    # You still have to write ${{ secrets.COVERITY_SCAN_TOKEN }} explicitly as
    # GitHub Actions are not allowed to access secrets directly.
    #
    # REQUIRED.
    token: ${{ secrets.COVERITY_SCAN_TOKEN }}

    # Where Coverity Scan should send notifications.
    #
    # The Coverity Scan tool requires this be set.
    #
    # If you don't want to write this in your config files, you can also use a
    # repository secret.  "COVERITY_SCAN_EMAIL" is a common name.  See the
    # previous "token" section for more information.
    #
    # REQUIRED.
    email: 'foo@example.com'

    # Which Coverity Scan language pack to download.
    #
    # May be "cxx", "java", "csharp", "javascript", or "other".
    #
    # See the Coverity Scan download page for possible values:
    # https://scan.coverity.com/download
    # The tab strip along the top lists the languages.
    #
    # NB: 'cxx' is used for both C & C++ code.
    #
    # Default: 'cxx'
    build_language: 'cxx'

    # Which Coverity Scan platform pack to download.
    #
    # See the Coverity Scan download page for possible values:
    # https://scan.coverity.com/download
    # The tab strip along the right side lists the platforms.
    #
    # Default: 'linux64'
    build_platform: ''

    # Command to pass to cov-build.
    #
    # Default: 'make'
    command: ''

    # Specify the working-directory of where to run commands.
    # https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#defaultsrun
    #
    # This directory will not be created automatically, it must exist already.
    #
    # Default: ${{ github.workspace }}
    working-directory: ''

    # (Informational) The source version being built.
    #
    # Default: ${{ github.sha }}
    version: ''

    # (Informational) A description for this particular build.
    #
    # Default: coverity-scan-action ${{ github.repository }} / ${{ github.ref }}
    description: ''
```

# Requirements

These tools need to be available.  The default Ubuntu Linux container provides
them already, so no extra work is needed there.  Other environments (e.g. macOS
& Windows) have not yet been tested.  Feedback welcome!

* [Bash](https://www.gnu.org/software/bash/): The shell used in steps.
* [POSIX sed](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/sed.html):
  Internal text transformation.
* [GNU tar](https://www.gnu.org/software/tar/): Unpacks Coverity tool.
* [GNU gzip](https://www.gnu.org/software/gzip/): Decompresses Coverity tool.
* [curl](https://curl.se/): Downloads Coverity tool & uploads results.
* Whatever Coverity Scan tool itself needs.

# FAQ

## How do I run configure/pre-build steps with this action?

Simply put, you don't!  This action specifically only has a single "build
command" because that's all the Coverity Scan tools accept.  If you want to
run `./configure` or `cmake` or something else before the `make`, add a step
to run those commands first.

A design limitation of GitHub Actions is that all of an action's output &
sub-steps are logged as a single discrete step, and diving down into each
sub-step is not clean.  So the more steps that run inside of this
`vapier/coverity-scan-action` step, the harder it is for you to parse your logs
when a failure occurs.

For example, you want to do:
```
...
jobs:
  ...
    steps:
    - uses: actions/checkout@v3
# Here is your dedicated configure/pre-build set of commands.
    - run: ./configure ...
# Then you can run coverity build.
    - uses: vapier/coverity-scan-action@v1
      with:
        ...
```

## Downloading cov-analysis.tar.gz fails with authentication errors!

If wget fails with `Username/Password Authentication Failed`, double check your
token and your project settings.  The token must match the Coverity Scan site
exactly, as must the project name.  Both of these are case sensitive.

Keep in mind that, while GitHub treats your project name case insensitively when
using git commands or browsing the web site, Coverity Scan does not.  So you
must use the exact same case that GitHub shows when you visit the project, and
as Coverity Scan shows it.

You can always copy & paste the wget command into your local terminal to check
both settings.

## I don't want to specify my e-mail address!

Unfortunately, this is required by Coverity Scan itself, not by this GitHub
Action.  If you try to submit results to Coverity Scan without an e-mail
address, it will reject the submission.

If you don't want to list your e-mail address in the config file, you can move
it to the repository secrets as `COVERITY_SCAN_EMAIL`, and then use
`email: ${{ secrets.COVERITY_SCAN_EMAIL }}` in your config file.  This will
prevent leakage to the wider internet, and avoid your e-mail address being
accidentally used when people fork your repository.

If you don't want to use a single person's e-mail address, you can always use
an alias or mailing list instead.  Setting up such a thing is way outside the
scope of this project though :).

## Who sees my code and/or results?

This action executes within the context of your project, so it will see your
code, but the results are only sent to Coverity Scan, and shown to you.  The
data is not sent anywhere else, so this action isn't stealing your data :).

Feel free to review or audit the [source code](./action.yml).  It fits on just
one page!

Of course, this is all independent of the settings on the Coverity Scan website
which has its own set of ACLs that you can control.

## Can I allow or block pull requests (PRs) from being scanned?

Sure, set the [`on` setting](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#on)
in your GitHub workflow to fire on whatever branch or event you want to include.
If you only want scans to run on pushes/merges to the main branch, then omit the
`pull_request` setting.  Or if you want to include PRs, then include the
setting.

## Do you like M&M's in your cookies?

Of course, [M&M's](https://en.wikipedia.org/wiki/M%26M%27s) cookies are great.
They're also great in brownies, especially
[all edges brownies](http://www.bakersedge.com/product_ebp.html).

However, [Smarties](https://en.wikipedia.org/wiki/Smarties) are an abomination
and you should be ashamed if you put them in either cookies or brownies.

### Are you a corporate shill?

I don't think I am.  Although if Mars or Nestle wanted to sponsor me, I wouldn't
say no.

# Migrating From Other CIs

## Coverity Scan Travis CI

Coverity Scan offers a `coverity_scan` addon:
https://scan.coverity.com/travis_ci

Converting from that Travis CI addon to this GitHub Action is fairly trivial.
Let's convert their example config file over.

The `[*]` lines aren't needed at all with the GitHub Action.

```
# Travis CI config.
    env:
      global:
        # COVERITY_SCAN_TOKEN
        # ** specific to your project **
[1]     - secure: "xxxx"

    addons:
[2]   coverity_scan:
        project:
[3]       name: my_github/my_project
[*]       version: 1.0
[*]       description: My Project
[4]     notification_email: scan_notifications@example.com
[5]     build_command_prepend: ./configure
[6]     build_command: make
[7]     branch_pattern: coverity_scan
```

```
# GitHub Action config.
    name: Coverity Scan

    on:
      push:
[7]     branches: [coverity_scan]

    jobs:
      # NB: "coverity" here can be anything you want.
      coverity:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v3
        - name: Configure
[5]       run: ./configure
[2]     - uses: vapier/coverity-scan-action@v1
          with:
[4]         email: vapier@gentoo.org
[3]         project: my_github/my_project
[1]         token: ${{ secrets.COVERITY_SCAN_TOKEN }}
[6]         command: make
```

# License

This project uses the [MIT License](LICENSE).
