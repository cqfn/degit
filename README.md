<img src="/logo.svg" width="92px"/>

DeGit is a decentralized [Git](https://git-scm.com/) projects hosting platform.
You can join by starting a node and pointing your browser
at `127.0.0.1`. Then, just work with it as if it was GitHub.
There is no central point of failure,
since the network of DeGit nodes is run by anonymous volunteers.

To start, simply do (it uses your `.ssh/id_rsa` for authentication):

```bash
$ gem install degit
$ degit run
```

In a few seconds you can open `https://localhost:8080` and enjoy
the system, which is very similar to GitHub. You can, of course, use
local Git repo, which is on-fly synchronized with other DeGit nodes.

## Motivation and Related Works

We are not the first who are thinking about a decentralized solution
for hosting and managing of Git repositories. There were a few similar products
created before (if you know anything else, please submit a pull request):

  * [GitChain](http://gitchain.org/) (abandoned in 2014)
  * [GitTorrent](https://github.com/cjb/GitTorrent) (abandoned in 2015)
  * [Drepo](https://www.drepo.io/) (abandoned in 2018)
  * [Radicle](https://github.com/radicle-dev) (read [this](https://outlierventures.io/wp-content/uploads/2019/11/Radicle-Diffusion-2019-1.pdf))
  * [git-issue](https://github.com/dspinellis/git-issue)
  * [git-ssb](https://scuttlebot.io/apis/community/git-ssb.html)
  * [git-dit](https://github.com/neithernut/git-dit)
  * [pando](https://github.com/pandonetwork/pando)
  * [mango](https://github.com/axic/mango) (abandoned in 2016, watch [this](https://www.youtube.com/watch?v=tU7_Yf45okc))
  * [ZeroNet](https://zeronet.io/) (not exactly Git, but relevant)

Even though [GitHub](https://github.com),
[GitLab](https://gitlab.com),
[BitBucket](https://bitbucket.com),
[Phabricator](https://phacility.com/phabricator/),
[SourceForge](https://sourceforge.net/),
[CodeCommit](https://console.aws.amazon.com/codecommit/home),
and
[Gitee](https://gitee.com) are great platforms,
they have three critical drawbacks:

  * They are [not](https://news.ycombinator.com/item?id=20499070) 100% reliable,
  * They [ban](https://medium.com/@catamphetamine/how-github-blocked-me-and-all-my-libraries-c32c61f061d3)
    users for
    [almost](https://medium.com/@hamed/github-blocked-my-account-and-they-think-im-developing-nuclear-weapons-e7e1fe62cb74)
    [no reason](https://en.wikipedia.org/wiki/Censorship_of_GitHub), and
  * They are under the [influence](https://techcrunch.com/2019/07/29/github-ban-sanctioned-countries/) of their local governments.

It seems that the need for a decentralized solution is obvious.
We believe that the community would enjoy having a platform
with the following features:

  * [Pull requests](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests);
  * Issues and milestones;
  * Stars and followers;
  * GitHub-like web user interface;
  * Entirely free for everybody;
  * Not owned by anyone;
  * Moderated by the board of deputies.

DeGit doesn't support private repositories, only public ones.

## How It Works?

The following principles are behind the architecture of DeGit:

  * `.degit` directory in `master` branch is used to keep meta information
  * Ownership of a repo is defined by public [RSA](https://en.wikipedia.org/wiki/RSA_%28cryptosystem%29) keys in `.degit`
  * Issues, PRs, comments, stars, etc. are regular files in `.degit`
  * Issues, PRs, and comments have hash codes instead of sequential IDs
  * Each node decides for itself which repositories to host
  * Give-and-take principle is in place: "The more you host for me, the more I host for you"
  * Conflicts are resolved through proof-of-availability (PoA) consensus
  * Neighbours-discovery protocol is similar to the one used in [Zold](https://blog.zold.io/2018/12/28/nodes-discovery-protocol.html)

Here is how it looks:

<img src="/diagram.svg" height="450px"/>

Two elements are part of the DeGit project: Dashboard and Hooks.

### Data Flow Explained

"Availability" is a non-negative integer assigned by a node to each of its neighbours.
The number goes up on every successful interaction with the neighbour. The
number goes double-down on each network failure or any other
non-logical error.

Here is how the data is propagated when you interact with Git on your laptop
(the same happens automatically behind the scene if you use UI in the browser):

  * You `git commit` your changes to your branches
  * You do `git push` to your `localhost`
  * On success, a built-in post-commit [hook](https://git-scm.com/docs/githooks) proceeds:
  * It `git fetch` from the first neighbour with the highest availability
  * It `git merge` if possible and all commits are signed correctly
  * It `git push` back to the neighbour

It is highly recommended to avoid pushing to the
same branch from a few nodes,
since it may lead to inability to merge and abandonded
(or lost) branches.

### Authorization and Authentication

A repository has a list of files in `.degit/permissions` directory. Each file
starts with a public RSA key and lists user IDs, permissions, etc.

On each `git push` event, post-commit hook goes through the list of added
commits and verifies permissions of each user. If any rule from
`.degit/permissions` is not respected, the entire `push` operation is rejected.

It is recommended to have at least two users with write access to the `master`
branch, in order to avoid losing access to the repo when
private RSA key is lost.

### Incentives

Unlike [Blockchain](https://en.wikipedia.org/wiki/Blockchain),
a full duplication of all database in all nodes is not
required for DeGit. Instead, if a few nodes have the data of a repository,
this may be enough for the majority of cases. Thus, each node tries to
host a limited number of repositories, according to the disc space available.

Also, each node maintains a list of repositories seen along with the addresses of their
hosting nodes. When a `git fetch` arrives for a repository that
the node doesn't have, it returns an error and a suggested list of nodes
to ask for this repo.

Thus, no monetary incentives are provided to node owners, but they are not
expected to contribute large computational or storage resources to the system
(like it happens in [Bitcoin](https://en.wikipedia.org/wiki/Bitcoin), for example).

### Anti-Spam

[Vandalism](https://en.wikipedia.org/wiki/Vandalism)
is possible through new issues and comments: they may be
submitted in large amounts. In order to fight against this, users may use
scripts (a concept very close to [smart contracts](https://en.wikipedia.org/wiki/Smart_contract))
to be executed on each post-commit hook to check
the validity of data submitted to `.degit`.

### Moderation

To be continued...

## How to Contribute?

Just give us a star and wait. We'll update this page soon.
