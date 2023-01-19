# Writing your own GitHub CLI Extensions

_with @vilmibm from the CLI team_

## me

```
 _          _   _        
| |        | | | |      |
| |     _  | | | |  __  |
|/ \   |/  |/  |/  /  \_|
|   |_/|__/|__/|__/\__/ o


```

I'm [@vilmibm](https://github.com/vilmibm), a senior engineer at GitHub. 

I've been here since 2018 and was a founding member of the CLI team. 

I love the terminal.

## The GitHub CLI 

The GitHub CLI, `gh`, brings GitHub into your terminal.

Marketing website:   https://cli.github.com
Public Repository:   **cli/cli**
Internal Repository: **github/cli**

It's an open source, flexible, resource light, and extensible way to interact with the GitHub.

`gh` can do many things, including:

- Create, list and update pull requests
- Create, list, and update issues
- Clone, fork, and view repositories
- Kick off or monitor workflow runs in Actions
- Interact with GitHub APIs
- Connect to and manage Codespaces

```
# List all the open issues in the cli/cli repository
gh -Rcli/cli issue list
```

## CLI Extensions

`gh` users can install third party commands published as `git` repositories.

Extensions can be small, one-off tools or large, complex CLI products. They can also be art projects. The world is your oyster.

They can be written in any language, though user experience is best when your extension can be pre-compiled into a single binary.

Some examples:

- `mislav/gh-branch` fuzzy-find switcher for git branches
- `github/gh-net` network bridging between a local machine and a codespace
- `vilmibm/gh-screensaver` ASCII animations to stare at while builds are running
- `dlvhdr/gh-dash` A rich TUI dashboard of your work on GitHub

## 0.1. Prerequisite: get gh installed and authenticated

### Installation

If you already have `gh` installed and authenticated, feel free to zone out for this part.

```
# MacOS
brew install gh

# Windows
winget install --id GitHub.cli
```

If you're on Linux, check out our [linux installation docs](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)

### Authentication

```
# Start the authentication wizard
gh auth login
```

### Test it out

```
gh api graphql -fquery='query { viewer { login } }'

# Should return something like:
#{
#  "data": {
#    "viewer": {
#      "login": "vilmibm"
#    }
#  }
#}
```

## 0.2. Prerequisite: Installing go

If you already have `go` installed, feel free to zone out for this part.

### Installing

Head to [the Go downloads page](https://go.dev/dl/). Download and run the appropriate installation media.

**_!!!_** You may need to close and re-open any terminals you have open for `go` to be added to your path.

### Verify that the installation worked

Create a new file in your home directory called `main.go` with the contents:

```go
package main

import "fmt"

func main() {
  fmt.Println("oh hey")
}
```

Try running the file:

```
go run ~/main.go
# oh hey
```

## 1. Finding extensions and installing extensions

Before we actually make a new extension, let's see what it looks like when an extension has been published.

Two commands help discover extensions:

```
# Print all installable extensions
gh ext search

# Interactively browse through extensions
gh ext browse
```

Let's install one by name:

```
gh ext install vilmibm/gh-screensaver
gh ext list
gh screensaver -sstarfield
```

By the end of this workshop, you'll know how to publish extensions that are discoverable and installable like this one.

## 2.0. Create a boilerplate extension

For the purposes of this workshop, we'll be recreating my sample CLI extension, [gh-ask](https://github.com/vilmibm/gh-ask).

This extension searches a repository's Discussions forum for threads relating to a search term.

In the end, it'll work like this:

```
gh ask -R cli/cli auth
Searching discussions in 'cli/cli' for 'auth'

gh auth login on linux doesn't let me do git push (i...  https://github.com/cli/cli/discussions/6866
Cannot log in via browser or personal access token       https://github.com/cli/cli/discussions/6858
API call failed: USER does not have the correct perm...  https://github.com/cli/cli/discussions/6814
```

In case you've installed my extension already, please run `gh ext remove ask` to avoid confusion.

First, run `gh`'s wizard for creating a new Go extension:

```
# In a directory where you want some source code to live:
gh ext create
```

1. The name of your extension is whatever a user would type to run it as a `gh` command. For this workshop, use the name `ask`.
2. Select **Go** as the answer to _What kind of extension?_

You should see a result like:

```
$ gh ext create
? Extension name: ask
? What kind of extension? Go
✓ Created directory gh-ask
✓ Initialized git repository
✓ Set up extension scaffolding
✓ Downloaded Go dependencies
✓ Built gh-ask binary

gh-ask is ready for development!

Next Steps
- run 'cd gh-ask; gh extension install .; gh ask' to see your new extension in action
- use 'go build && gh ask' to see changes in your code as you develop
- commit and use 'gh repo create' to share your extension with others

For more information on writing extensions:
https://docs.github.com/github-cli/github-cli/creating-github-cli-extensions
```

## 2.1. Create a repository for the extension

`gh` just created a local `git` repository for you. It has no commits and does not yet exist on GitHub. To save on confusion later, we'll go ahead and finalize this repository before we work on it.

```
cd gh-ask
git add .
git commit -m'initial commit'
gh repo create
```

Running `gh repo create` starts an interactive wizard; use the following answers:

- `? What would you like to do?` Push an existing local repository to GitHub
- `? Path to local repository` .
- `? Repository name` gh-ask
- `? Repository owner` <your name>
- `? Description` gh-ask
- `? Visibility` Private
- `? Add a remote?` Yes
- `? What should the new remote be called?` origin

Your output should look something like:

```
? What would you like to do? Push an existing local repository to GitHub
? Path to local repository .
? Repository name gh-ask
? Repository owner vilmibm
? Description gh-ask
? Visibility Private
✓ Created repository vilmibm/gh-ask on GitHub
? Add a remote? Yes
? What should the new remote be called? origin
✓ Added remote git@github.com:vilmibm/gh-ask.git
? Would you like to push commits from the current branch to "origin"? Yes
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 24 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 3.42 KiB | 3.42 MiB/s, done.
Total 9 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:vilmibm/gh-ask.git
 * [new branch]      HEAD -> trunk
Branch 'trunk' set up to track remote branch 'trunk' from 'origin'.
✓ Pushed commits to git@github.com:vilmibm/gh-ask.git
```

## 2.2. Run the new extension

It doesn't do much yet, but it's already possible to run this new extension.

The output should look something like:

```
go run .
#hi world, this is the gh-ask extension!
#running as vilmibm
```

## 3.0. Writing the extension

TODO

## 3.1. Basic structure

TODO

## 3.2. Supporting flags

TODO

## 3.3. Supporting arguments

TODO

## 3.4. API calls

TODO

## 3.5. Output formatting

TODO

## 3.6. Supporting JSON

TODO

## 3.7. Opening browsers

TODO

## 4.0. Publishing releases

TODO

## 5.0. Wrapping up

TODO
