# SPEC-0 Versions Action

This repository contains a GitHub Action to generate the files required for the SPEC-0 documentation.

## Using the action

```yaml
name: Generate spec-zero data

on:
  push:
    branches:
      - main

jobs:
  devstats-query:
    runs-on: ubuntu-latest
    steps:
      - uses: scientific-python/spec-zero-tools@main
```

The above would produce an artifact named `spec-zero-versions`, the following files: `schedule.yaml`,`schedule.md` and `chart.md`.

To help projects stay compliant with SPEC-0, we provide a `schedule.json` file that can be used by CI systems to determine new version boundaries.
The structure of the file is as follows:

```json
[
  {
    "start_date": "iso8601_timestamp",
    "packages": {
      "package_name": "version"
    }
  }
]
```

All information in the json file is in a string format that should be easy to use.
The date is the first timestamp of the relevant quarter.
Thus a workflow for using this file could be:

1. Fetch `schedule.json`
2. Determine maximum date that is smaller than current date
3. Update packages listed with new minimum versions

You can obtain the new versions you should set by using this `jq` expression:

```sh
jq 'map(select(.start_date |fromdateiso8601 |tonumber  < now))| sort_by("start_date") | reverse | .[0].packages ' schedule.json
```

If you use a package manager like pixi you could update the dependencies with a bash script like this (untested):

```sh
curl -Ls -o schedule.json https://raw.githubusercontent.com/scientific-python/specs/main/spec-0000/schedule.json
for line in $(jq 'map(select(.start_date |fromdateiso8601 |tonumber  < now))| sort_by("start_date") | reverse | .[0].packages | to_entries | map(.key + ":" + .value)[]' --raw-output schedule.json); do
  package=$(echo "$line" | cut -d ':' -f 1)
  version=$(echo "$line" | cut -d ':' -f 2)
  if pixi list -x "^$package" &>/dev/null| grep "No packages" -q; then
    pixi add "$package>=$version";
  fi
done
```
