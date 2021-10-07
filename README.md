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
    strategy:
      matrix:
        os: [ubuntu-latest]
        cc: [gcc]
    runs-on: ${{ matrix.os }}
    env:
      CC: ${{ matrix.cc }}
    steps:
    - uses: actions/checkout@v2
    - uses: vapier/coverity-scan-action@v1
      with:
        token: ${{ secrets.COVERITY_SCAN_TOKEN }}
```

Make sure to define `COVERITY_SCAN_TOKEN` in your
[project's secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets).

# Usage

```yaml
- uses: vapier/coverity-scan-action@v0
  with:
    # Project name in Coverity Scan.
    #
    # This should be as it appears on the Coverity Scan website.
    # Find it in your dashboard:
    # https://scan.coverity.com/dashboard
    #
    # For example, a GitHub project will look like "gentoo/pax-utils".
    #
    # Default: ${{ github.repository }}
    project: ''

    # Secret project token for accessing this project in Coverity Scan.
    #
    # Find this in the project's "Project Settings" tab under "Project token".
    #
    # This value should not be specified in the yaml file directly.  Instead it
    # should be set in your repositories secrets.  "COVERITY_SCAN_TOKEN" is a
    # common name here.
    # https://docs.github.com/en/actions/security-guides/encrypted-secrets
    #
    # You still have to list ${{ secrets.COVERITY_SCAN_TOKEN }} explicitly as
    # GitHub Actions are not allowed to access secrets directly.
    #
    # REQUIRED.
    token: ${{ secrets.COVERITY_SCAN_TOKEN }}

    # Where Coverity Scan should send notifications.
    #
    # The Coverity Scan tool requires this be set.
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

    # (Informational) The source version being built.
    #
    # Default: ${{ github.sha }}
    version: ''

    # (Informational) A description for this particular build.
    #
    # Default: coverity-scan-action ${{ github.repository }} / ${{ github.ref }}
    description: ''
```

# License

This project uses the [MIT License](LICENSE).
