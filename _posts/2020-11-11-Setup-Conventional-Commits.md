---
layout: post
section-type: post
title: 'Setup and Customize Conventional Commits and Semantic Versioning'
date: 2020-11-16 17:30:22.000000000 +00:00
categories: Code
post-preview-words: Learn how to setup conventional commits and semantic versioning...
---

In an earlier [article](https://evanjmg.com/code/2020/11/08/Automate-reporting-with-changelogs.html), I went through the importance and use-cases for semantic versioning and conventional commits. Now, I'll go through setting up the system and customizing it.

### Conventional Commits

Conventional commits enable a team to set strict rules for commits and validate the commits before they are committed or merged.

There are a couple of conventional commit standards out there on the Internet. Most of the libraries out there are npm (NodeJS) libraries, written in JavaScript, and some specifications that function as guidelines.  Some of the following are:

- [Commitizen CLI](https://github.com/commitizen/cz-cli) (npm library) is the most popular conventional commit generation tool. It includes changelog generation from the formatted commits when releasing.
- [Conventional Changelog](https://github.com/conventional-changelog/conventional-changelog) is a changelog generation tool that commitizen uses under the hood. Allows one to customize the changelog generation and other bits in the process.
- [Conventional Commits Standards](https://www.conventionalcommits.org/en/v1.0.0/) (specification) is useful resource to understand how the formatting of conventional commits works.
- [CommitLint](https://github.com/conventional-changelog/commitlint#readme) validates commits to see if matches the standard formatting of a conventional commit.

In this example, I'm going to use Commitizen and Commitlint to glue everything together. Commitizen simply prompts the user to select options or fill in parts of a commit. This prevents the user from making a colon typo etc.

First install,
```
 npm i commitizen @commitlint/cli husky @commitlint/config-conventional --save-dev
```

Husky allows one to validate the commit on the final step of the pre-commit process. Add the following to the package.json to setup the pre-commit process.

```
"husky": {
  "hooks": {
    "pre-commit": "npm run lint",
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```
Here, Commitlint reads the git commit metadata output from the pre-commit step. If the commit matches the (standard format](https://www.conventionalcommits.org/en/v1.0.0/) of a conventional commit, then the commit will fail. So this prevents developers from putting random commit messages like git commit -m "Almost done!".

And you will also need to set a lint config, like so:

```
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

*Making a commit*

Instead of committing, developers should use git cz or cz. To set this up, add the following to your package.json
```
"scripts": {
  "commit": "cz"
},
"config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }

```

Now, I can run `npm run commit` or run git-cz. I can then get a series of prompts that I can customize by adding more configuration values [here](https://www.npmjs.com/package/cz-conventional-changelog). If I want to change the questions, I can use a different changelog plugin. For instance, I can check out different forks and fork my own to match my development workflow (e.g [Ryanair's fork](https://github.com/Ryanair/cz-conventional-changelog)).

### Semantic Versioning

Semantic versioning is a versioning standard that allows one to increment a release automatically based on a set of standardized (conventional) commits. The introduction of npmjs (node package manager) forced many JavaScript library maintainers to adopt semantic versioning. These rules align with conventional commits and allow one to generate changelogs as well. In doing so, the community created the following libraries to generate their versioning:

- [standard-version](https://www.npmjs.com/package/standard-version) is a simple versioning tool for creating releases locally and making a PR. I recommend starting here when discussing releases and conventional commits with your team.
- [semantic-release](https://www.npmjs.com/semantic-release) is a more advanced tool that works with automated CI systems (Jenkins, Travis CI, etc). If you have a clear understanding of how and when you want to make releases, then this is the best tool to use. If your team is still learning, standard-version is a simpler tool.

To setup standard-version, simply:
```
npm i standard-version --save-dev
```

and add this line to the scripts section of the package.json:

```
 "scripts": {
   "release": "standard-version"
 }
```

Before releasing, ensure to create a chore commit of this semantic release update. When you have a group of commits you want to tag and release simply `npm run release`

The version will depend on whether there is a feature, bug, or breaking change in the set of commits.

Also notice that a CHANGELOG.md is generated and package.json version is also updated.

The changelog links and more settings can be updated and changed by the .versionrc.json file. The most useful items to change are the commit and issue url format:
```
{
  "commitUrlFormat": "https://bitbucket.org/YOUR_COMPANY/{{repository}}/commits/{{hash}}",
  "issueUrlFormat": "https://jira.atlassian.net/browse/{{id}}"
}
```

When I delete the release tags on the local change, `git reset --hard HEAD^` on the release commit, and run `npm run release` again then the urls will be updated in the CHANGELOG.md

Hope you find this useful! Let me know if you find any cool things I missed!

Source code is [here](https://github.com/evanjmg/changelog-example/).





