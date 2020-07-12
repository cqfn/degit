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

  * Pull requests, stars, and followers,
  * GitHub-like web user interface,
  * Entirely free for public and private repositories,
  * Not owned by anyone,
  * Moderated by the board of deputies.

## How It Works?

The following principles are behind the architecture of DeGit:

  * An author is the owner of a node, authenticated by his/her [RSA key](https://en.wikipedia.org/wiki/RSA_%28cryptosystem%29)
  * Issues, PRs, comments, stars, etc. are regular files in `.degit` directory in `master`
  * Issues, PRs, and comments have hash codes instead of sequential IDs
  * Each node decides for itself which repositories to host
  * Give-and-take principle is in place: "The more you host for me, the more I host for you"
  * Commits are announced to neighbour nodes, which they can `git pull` later if they want
  * Conflicts are resolved through DeGit Consensus Algorithm (see below)
  * Neighbours-discovery protocol is similar to the one used in [Zold](https://blog.zold.io/2018/12/28/nodes-discovery-protocol.html)
  * Nodes communicate through HTTP RESTful interfaces

### Proof of Availability

"Availability" is a positive number assigned by a node to each of its neighbours.
The number goes up on every successful interaction with the neighbour. The
number goes double-down on each failure due to network failure or any other
not-logical error.

DeGit Consensus Algorithm based on PoA:

  * A branch dominates during [merge](https://git-scm.com/docs/git-merge) if the providing node is more _available_
  * The _availability_ of neighbours is subjectively judged by each node
  * [Commits](https://git-scm.com/docs/git-commit) from less _available_ branches are ignored during merge
  * The _availability_ of itself is configurable (either MAX or MIN)

### Data Flow Explained

<img src="http://www.plantuml.com/plantuml/proxy?src=https://raw.github.com/yegor256/degit/master/uml/data-flow.uml"/>

Here is how the data is propagated when you interact with Git on your laptop
(the same happens automatically behind the scene if you use UI in the browser):

  * You `git commit` your changes to your branches
  * You do `git push` to your `localhost`
  * On success, a built-in post-commit [hook](https://git-scm.com/docs/githooks) pushes them to your neighbour nodes
  * Some neighbours break the connection and ignore the data
  * Others attempt to merge the coming data with their local repositories
  * They resolve conflicts according to the Consensus Algorithm (MAX)

This is what happens on a node:

  * New commits arrive from another node (of the client)
  * We `git merge` them to the existing repository
  * Conflicts are resolved according to the Consensus Algorithm (MAX)
  * We `git pull` all neighbours
  * Conflicts are resolved according to the Consensus Algorithm (MIN)

It is highly recommended to avoid pushing to the
same branch from a few nodes,
since it may lead to inability to merge and abandonded
(or lost) branches.

## How to Contribute?

Just give us a star and wait. We'll update this page soon.
