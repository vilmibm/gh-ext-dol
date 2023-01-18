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

```
# In a directory where you want some source code to live:
gh ext create
```

TODO

## 2.1. Create a repository for the extension

TODO

##
