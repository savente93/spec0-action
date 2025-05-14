# SPEC 0 Version Action

This repository contains a GitHub Action to generate the

### Drop Schedule

Below is an auto generated schedule with recommended dates for dropping support. We suggest that the next release in a given quarter is
considered as the one removing support for a given item.

You may want to delay the removal of support of an older Python version until your package fully works on the newly released Python, thus keeping the number of supported minor versions of Python the same for your package.

{{< include-md "schedule.md" >}}

### Automatically updating dependencies

To help projects stay compliant with this spec, we additionally provide a `schedule.json` file that can be used by CI systems to deterime new version boundaries. The structure of the file is as follows:

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

All information in the json file is in a string format that should be easy to use. The date is the first timestamp of the relevant quarter. Thus a workflow for using this file could be:

1. fetch `schedule.json`
2. determine maximum date that is smaller than current date
3. update packages listed with new minimum versions

You can obtain the new versions you should set by using this `jq` expression:

```sh

jq 'map(select(.start_date |fromdateiso8601 |tonumber  < now))| sort_by("start_date") | reverse | .[0].packages ' schedule.json

```

If you use a package manager like pixi you could update the dependencies with a bash script like this:

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
