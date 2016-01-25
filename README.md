# gradle-android-git-version
[![Build Status](https://api.travis-ci.org/gladed/gradle-android-git-version.svg)](https://travis-ci.org/gladed/gradle-android-git-version)

A gradle plugin to calculate Android-friendly version names and codes from git tags. If you are tired of manually updating your Android build files for each release, or generating builds that you can't trace back to code, then this plugin is for you!

## Usage

Add the plugin the top of your `app/build.gradle` (or equivalent):
```groovy
plugins {
    id 'com.gladed.androidgitversion' version '0.2.7'
}
```

Set `versionName` and `versionCode` from plugin results:
```groovy
android {
    // ...
    defaultConfig {
        // ...
        versionName androidGitVersion.name()
        versionCode androidGitVersion.code()
```

Use a git tag to specify your version number (see [Semantic Versioning](http://semver.org))
```bash
$ git tag 1.2.3
$ gradle --quiet androidGitVersion
androidGitVersion.name	1.2.3
androidGitVersion.code	1002003
```

Any suffix after the version number in the tag (such as `-release4` in `1.2.3-release4`) is included in the version name, but is ignored when generating the version code.

## Intermediate Versions

For builds from commits that are not explicitly tagged, `name()` will return a build of this form:

`1.2.3-2-93411ff-fix_issue5-dirty`

The components in the example above are as follows:

| 1.2.3 | -2 | -93411ff | -fix_issue5 | -dirty |
| --- | --- | --- | --- | --- |
| Most recent tag | Number of commits since tag | Commit prefix | Branch name, if branch is not listed in `hideBranches` | Present only there are uncommitted changes |

## Version Codes

Version codes are calculated relative to the most recent tag.

The code is generated by combining each version part with a multiplier. Unspecified version parts are assumed to be zero even if not supplied. So version 1.22 becomes 1022000, while version 2.4.11 becomes 2004011. This allows successive versions to have incrementing version codes.

Intermediate versions do *not* produce a new code. The code only advances when a new version tag is specified.

You can configure this behavior with `multipler` and `parts` properties, but be warned that changing the version code scheme for a released Android project can cause problems if your new version code does not [increase monotonically](http://developer.android.com/tools/publishing/versioning.html).

## Methods

`name()` returns the current version name.

`code()` returns the current version code.

`flush()` flushes the internal cache of information about the git repo, in the event you have a gradle task that makes changes.

# Tasks

`androidGitVersion` prints the name and code, as shown above.

`androidGitVersionName` prints only the name.

`androidGitVersionCode` prints only the code.

## Configuration Properties

An `androidGitVersion` block in your project's `build.gradle` file can supply optional properties to configure this plugin's behavior, e.g.:

```groovy
androidGitVersion {
    prefix 'lib-'
    onlyIn 'my-library'
    multiplier 10000
    parts 2
    baseCode 2000
    hideBranches = [ 'develop' ]
}
```

### prefix (string)
Set a tag prefix to indicate that relevant version tags will start with the specified string. For example, with `prefix 'lib'`, a tags like `lib-1.5` will be found while a tag like `1.0` or `app-2.4.2` will be ignored.

The default for prefix is `''` which matches all numeric version tags.

### onlyIn (string)
Set the onlyIn path to indicate a path within your project. Commits that change files in this path will count, while other commits will not. This is useful when building a versioned library from a git project containing other projects (apps and other libraries).

For example, consider this directory tree:
```
+-- my-app/
    +-- .git/
    +-- build.gradle
    +-- app/
    |   +-- build.gradle
    |   +-- src/
    +-- lib/
        +-- build.gradle
        +-- src/
```
If a commit is tagged with `1.0.1`, and `my-app/lib/build.gradle` is configured with `onlyIn 'lib'`, then commits that change `my-app/build.gradle` or `my-app/app/src` will not affect the version name or code generated from `my-app/lib/build.gradle`.

The default onlyIn path is `''`, which includes all commits that change files.

### multiplier (int)
Changes the multipler used to combine each part of the version number.

For example, if you want version 1.2.3 to have a version code of 100020003 (allowing for 9999 patch increments), use `multiplier 10000`.

Use caution when increasing this value, as the maximum version code is 2147483647 (the maximum integer).

The default multiplier is 1000.

### parts (int)
Changes the assumed number of parts.

For example, if you know your product will only ever use two version number parts (1.2) then use `parts 2`.

Use caution when increasing this value, as the maximum version code is 2147483647 (the maximum integer).

The default number of parts is 3.

### baseCode (int)
A base version code added to all generated version codes. Use this when you have already released a version with a code, and don't want to go backwards.

The default baseCode is 0.

### hideBranches (list of strings)

A list of branches which should *not* be added to builds for intermediate (non-tagged) commits. This will result in somewhat cleaner intermediate version names.

The default hideBranches are `[ 'master', 'release' ]`.

## License

All code here is Copyright 2015 by Glade Diviney, and licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0).
