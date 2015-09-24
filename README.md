# Octop

Octop provides a minimal higher-level wrapper around git's [plumbing commands](http://git-scm.com/book/en/Git-Internals-Plumbing-and-Porcelain), exposing an API for manipulating GitHub repositories on the file level. It is being developed in the context of [Prose](http://prose.io), a content editor for GitHub.

This repo is now officially maintained by [WeFlex](https://github.com/weflex).

> Note: this is a fork from [michael/github](https://github.com/michael/github), thanks to DevelopmentSeed.

## Installation

Either grab `octop` from this repo or install via NPM:

```
npm install octop --save
```

## Usage

Create a Github instance.

```js
var github = new Github({
  username: "YOU_USER",
  password: "YOUR_PASSWORD",
  auth: "basic"
});
```

Or if you prefer OAuth, it looks like this:

```js
var github = new Github({
  token: "OAUTH_TOKEN",
  auth: "oauth"
});
```

You can use either:
* Authorised App Tokens (via client/secret pairs), used for bigger applications, created in web-flows/on the fly
* Personal Access Tokens (simpler to set up), used on command lines, scripts etc, created in GitHub web UI

See these pages for more info:

[Creating an access token for command-line use](https://help.github.com/articles/creating-an-access-token-for-command-line-use)

[Github API OAuth Overview] (http://developer.github.com/v3/oauth)

Enterprise Github instances may be specified using the `apiUrl` option:

```js
var github = new Github({
  apiUrl: "https://serverName/api/v3",
  ...
});
```

## Repository API


```js
var repo = github.getRepo(username, reponame);
```

Show repository information

```js
repo.show(function(err, repo) {});
```

Delete a repository

```js
repo.deleteRepo(function(err, res) {});
```

Get contents at a particular path in a particular branch.

```js
repo.contents(branch, "path/to/dir", function(err, contents) {});
```

Fork repository. This operation runs asynchronously. You may want to poll for `repo.contents` until the forked repo is ready.

```js
repo.fork(function(err) {});
```

Create new branch for repo. You can omit oldBranchName to default to "master".

```js
repo.branch(oldBranchName, newBranchName, function(err) {});
```

List Pull Requests.

```js
var state = 'open'; //or 'closed', or 'all'
repo.listPulls(state, function(err, pullRequests) {});
```

Get details of a Pull Request.

```js
var pullRequestID = 123;
repo.getPull(pullRequestID, function(err, pullRequestInfo) {});
```

Create Pull Request.

```js
var pull = {
  title: message,
  body: "This pull request has been automatically generated by Prose.io.",
  base: "gh-pages",
  head: "michael" + ":" + "prose-patch"
};
repo.createPullRequest(pull, function(err, pullRequest) {});
```

Retrieve all available branches (aka heads) of a repository.

```js
repo.listBranches(function(err, branches) {});
```

Store contents at a certain path, where files that don't yet exist are created on the fly.

```js
repo.write('master', 'path/to/file', 'YOUR_NEW_CONTENTS', 'YOUR_COMMIT_MESSAGE', function(err) {});
```

Not only can you can write files, you can of course read them.

```js
repo.read('master', 'path/to/file', function(err, data) {});
```

Move a file from A to B.

```js
repo.move('master', 'path/to/file', 'path/to/new_file', function(err) {});
```

Remove a file.

```js
repo.remove('master', 'path/to/file', function(err) {});
```

Get information about a particular commit.

```js
repo.getCommit('master', sha, function(err, commit) {});
```

Exploring files of a repository is easy too by accessing the top level tree object.

```js
repo.getTree('master', function(err, tree) {});
```

If you want to access all blobs and trees recursively, you can add `?recursive=true`.

```js
repo.getTree('master?recursive=true', function(err, tree) {});
```

Given a filepath, retrieve the reference blob or tree sha.

```js
repo.getSha('master', '/path/to/file', function(err, sha) {});
```

For a given reference, get the corresponding commit sha.

```js
repo.getRef('heads/master', function(err, sha) {});
```

Create a new reference.

```js
var refSpec = {
  "ref": "refs/heads/my-new-branch-name",
  "sha": "827efc6d56897b048c772eb4087f854f46256132"
};
repo.createRef(refSpec, function(err) {});
```

Delete a reference.

```js
repo.deleteRef('heads/gh-pages', function(err) {});
```

Get contributors list with additions, deletions, and commit counts.

```js
repo.contributors(function(err, data) {});
```

## User API


```js
var user = github.getUser();
```

List all repositories of the authenticated user, including private repositories and repositories in which the user is a collaborator and not an owner.

```js
user.repos(function(err, repos) {});
```

List organizations the autenticated user belongs to.

```js
user.orgs(function(err, orgs) {});
```

List authenticated user's gists.

```js
user.gists(function(err, gists) {});
```

List unread notifications for the authenticated user.

```js
user.notifications(function(err, notifications) {});
```

Show user information for a particular username. Also works for organizations.

```js
user.show(username, function(err, user) {});
```

List public repositories for a particular user.

```js
user.userRepos(username, function(err, repos) {});
```

Create a new repo for the authenticated user

```js
user.createRepo({"name": "test"}, function(err, res) {});
```
Repo description, homepage, private/public can also be set.
For a full list of options see the docs [here](https://developer.github.com/v3/repos/#create)


List repositories for a particular organization. Includes private repositories if you are authorized.

```js
user.orgRepos(orgname, function(err, repos) {});
```

List all gists of a particular user. If username is ommitted gists of the current authenticated user are returned.

```js
user.userGists(username, function(err, gists) {});
```

## Gist API

```js
var gist = github.getGist(3165654);
```

Read the contents of a Gist.

```js
gist.read(function(err, gist) {

});
```

Updating the contents of a Gist. Please consult the documentation on [GitHub](http://developer.github.com/v3/gists/).

```js
var delta = {
  "description": "the description for this gist",
  "files": {
    "file1.txt": {
      "content": "updated file contents"
    },
    "old_name.txt": {
      "filename": "new_name.txt",
      "content": "modified contents"
    },
    "new_file.txt": {
      "content": "a new file"
    },
    "delete_this_file.txt": null
  }
};

gist.update(delta, function(err, gist) {

});
```
## Issues API

```js
var issues = github.getIssues(username, reponame);
```

To read all the issues of a given repository

```js
issues.list(options, function(err, issues) {});
```

##Setup

Github.js has the following dependency:

- btoa (included in modern browsers, an npm module is included in package.json for node)

## Compatibility

[![browser support](https://ci.testling.com/darvin/github.png)](https://ci.testling.com/darvin/github)
