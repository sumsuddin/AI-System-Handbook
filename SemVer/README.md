# Semantic Versioning

* A [blog](https://threedots.tech/post/automatic-semantic-versioning-in-gitlab-ci/) goes through simplified [semver](https://github.com/fsaintjacques/semver-tool) automation.
    * Simple, easy to understand and gives full control
    * No installation required if bash script version is used
    * For small projects this solution might be enough
    * A sample example is shown below
* A [blog](https://semaphoreci.com/blog/semantic-versioning-cicd) on using [semantic-release](https://github.com/semantic-release/semantic-release) which is the most popular and feature rich solution as of now.
    * It uses a fixed commit pattern
    * Needs Node js and other related dependencies to be installed
    * Suitable for large projects when basic scripts can't do the job


### A Simple Semantic Versioning Solution:

Install [semver](https://github.com/fsaintjacques/semver-tool) on your system which is responsible for version bumps

``` bash
# Download the script and save it to /usr/local/bin
wget -O /usr/local/bin/semver \
  https://raw.githubusercontent.com/fsaintjacques/semver-tool/master/src/semver

# Make script executable
chmod +x /usr/local/bin/semver

# Prove it works
semver --version
# semver: 3.4.0

```

Following is a custom logic for automatic semver. 

* ```$ nano /usr/local/bin/autosemver ```
* Paste the following bash script and save the file
* ```$ chmod +x /usr/local/bin/autosemver```

``` bash
#!/bin/bash

# Get the latest tag
latest_tag=$(git describe --tags --abbrev=0)

# Determine the type of changes since the last release
changes=$(git log $latest_tag..HEAD --oneline)

if [ -z "$changes" ]; then
    echo "No changes detected."
    exit
fi

echo $changes

major_changes=$(echo "$changes" | grep -i "BREAKING CHANGE")
minor_changes=$(echo "$changes" | grep -i "feat")
patch_changes=$(echo "$changes" | grep -i "fix")

# Bump version based on changes
if [ -n "$major_changes" ]; then
    bump="major"
elif [ -n "$minor_changes" ]; then
    bump="minor"
else
    bump="patch"
fi

# Get current version
current_version=$(echo $latest_tag)

# Perform version bump
new_version=$(semver bump $bump $current_version)

# Update version in project files
sed -i "s/$current_version/$new_version/g" version.txt

# Commit and tag the new version
git commit -a -m "chore(release): $new_version" -m "This Release Includes Following Changes:" -m "$changes"
git tag $new_version
git push origin $new_version

```

Usage of the above script:

* Move to the repository root directory for which we need semver
* Manually create the first git tag as 0.0.0 (if none already available)
* Create a version.txt file with the initial version text in a line
* Simply run ```$ autosemver```
* The logic in the above script can be customized as necessary
* Can add the script to the `CI/CD` pipeline to automate the process when A PR is merged to the `master` branch

**NOTE:** The commit messages should follow the pattern to identify the major, minor or patch related changes. Also, when `autosemver` is executed it changes the `version.txt`, commits the changes and creates a new updated tag automatically. Another thing to keep in mind that the operation will not recover if partially failed.