# Writing your own GitHub CLI Extensions

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

- Marketing website:   https://cli.github.com
- Public Repository:   https://github.com/cli/cli
- Internal Repository: https://github.com/github/cli

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
gh --repo cli/cli issue list
```

## The GitHub CLI

For many GitHub users, `gh` is _the_ point of entry for our company's products. We've heard directly from users that:

- in low-bandwidth areas, `gh` means that GitHub resources can load for them
- in areas with unreliable internet, `gh` helps maintains a consistent session
- for users who rely on screen readers, `gh` is much easier to navigate than github.com

For these reasons, I have an ulterior motive in doing this presentation:

**Convincing hubbers to ship features as GitHub CLI extensions**

`gh` is not just a toy for power users or an API wrapper; it's a way to reach huge demographics of developers with software development tooling that can make their work go more smoothly, both supporting GitHub's mission and creating superfans.

## CLI Extensions

`gh` users can install third party commands published as `git` repositories.

Extensions can be small, one-off tools or large, complex CLI products. They can also be art projects. The world is your oyster.

They can be written in any language, though user experience is best when your extension can be pre-compiled into a single binary.

Some examples:

- `mislav/gh-branch` fuzzy-find switcher for git branches
- `github/gh-net` network bridging between a local machine and a codespace
- `vilmibm/gh-screensaver` ASCII animations to stare at while builds are running
- `dlvhdr/gh-dash` A rich TUI dashboard of your work on GitHub

## Prerequisites

### Get gh installed and authenticated

If you already have `gh` installed and authenticated, feel free to zone out for this part.

```
# MacOS
brew install gh

# Windows
winget install --id GitHub.cli
```

If you're on Linux, check out our [linux installation docs](https://github.com/cli/cli/blob/trunk/docs/install_linux.md).

Once installed, authentication `gh`:

```
# Start the authentication wizard
gh auth login
```

and then test it out:

```
gh api graphql -fquery='query { viewer { login } }'

{
  "data": {
    "viewer": {
      "login": "vilmibm"
    }
  }
}
```

### Installing go

If you already have `go` installed, feel free to zone out for this part.

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
oh hey
```

## Finding extensions and installing extensions

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

By the end of this guide, you'll know how to publish extensions that are discoverable and installable like this one.

## Set up a new extension

### Create a boilerplate extension

For the purposes of this workshop, we'll be recreating my sample CLI extension, [gh-ask](https://github.com/vilmibm/gh-ask).

This extension searches a repository's Discussions forum for threads relating to a search term.

In the end, it'll work like this:

```
gh ask --repo cli/cli auth
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

### Create a repository for the extension

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

### Run the new extension

It doesn't do much yet, but it's already possible to run this new extension.

The output should look something like:

```
go run .
hi world, this is the gh-ask extension!
running as vilmibm
```

## Writing the extension

We'll be making heavy use of a library called [go-gh](https://github.com/cli/go-gh) to write our extension. The CLI team has extracted a lot of code from `gh` and put it in an external library so extension authors can take advantage of our work. Using `go-gh` also helps make your extension behave consistently with `gh`.

**!!!** Sometimes after updating imports in your Go code, you'll get an error about needing to run `go mod tidy`. If this happens, run `go mod tidy`.

### Basic structure

Open up main.go and delete everything except `package main` at the top. Then, fill in some new code:

```go
package main

import (
  "fmt"
  "os"
)

func main() {
  if err := cli(); err != nil {
    fmt.Fprintf(os.Stderr, "gh-ask failed: %s\n", err.Error())
    os.Exit(1)
  }
}

func cli() error {
  // The meat of our extension will go here
  return nil
}
```

### User input: repository and search term

`gh-ask` should either determine what repository to query based on the terminal's current directory or respect a `--repo` flag (like `gh` does).

`go-gh` helps us do this.

- `gh.CurrentRepository()` can tell us what GitHub repository is represented by the current directory
- `repository.Parse(nameWithOwner)` can parse a user-provided repository name

We'll start by adding some imports to the top of our file:

```go
package main

import (
  "errors"
  "flag"
  "fmt"
  "os"
  "strings"

  "github.com/cli/go-gh"
  "github.com/cli/go-gh/pkg/repository"
)
```

and then support a `--repo` flag and positional argument in `cli()`:

```go
package main
// import ...
// func main() ...
func cli() error {
  repoOverride := flag.String("repo", "", "Repository to query. Current directory used by default.")
  flag.Parse()

  if len(flag.Args()) < 1 {
    return errors.New("search term required")
  }
  searchTerm := strings.Join(flag.Args(), " ")

  var repo repository.Repository
  var err error

  if *repoOverride == "" {
    repo, err = gh.CurrentRepository()
  } else {
    repo, err = repository.Parse(*repoOverride)
  }
  if err != nil {
    return fmt.Errorf("could not determine repository to query: %w", err)
  }

  fmt.Printf("going to search for %s in %s/%s\n", searchTerm, repo.Owner(), repo.Name())

  return nil
}
```

Running your extension should look something like:

```
go run . auth
going to search for auth in vilmibm/gh-ask

go run . --repo cli/cli auth
going to search for auth in cli/cli
```

In the first invocation, the result of `gh.CurrentRepository()` is used. In the second, the value of `--repo` is respected.

### Calling a GitHub API

In this step, we'll modify our code to use the GitHub GraphQL API. We'll use a
combination of `go-gh`'s GraphQL client and a supporting graphql library called
`shurcool/graphql`.

First, add a new import:

```go
import (
  // other imports
  graphql "github.com/cli/shurcooL-graphql"
)
```

### Adding GraphQL query structs

We'll need to add two type declarations to our code. One describes a discussion and the other describes the shape of our overall GraphQL query.

At the top level of your code, add:

```go
type discussion struct {
  Title string `json:"title"`
  URL   string `json:"url"`
  Body  string `json:"body"`
}

type gqlQuery struct {
  Repository struct {
    HasDiscussionsEnabled bool
    Discussions struct {
      Edges []struct {
        Node discussion
      }
    } `graphql:"discussions(first: 100)"`
  } `graphql:"repository(owner: $owner, name: $name)"`
}
```

### Running the query

Now that we have types to support our use of GraphQL, it's time to make an API call and process the result.

```go
package main
// import ...
// func main() ...
// type discussion...
// type gqlQuery...

func cli() error {
  // input processing...
  client, err := gh.GQLClient(nil)
  if err != nil {
    return fmt.Errorf("could not make graphql client: %w", err)
  }

  variables := map[string]interface{}{
    "owner": graphql.String(repo.Owner()),
    "name": graphql.String(repo.Name()),
  }

  var query gqlQuery

  if err = client.Query("DiscussionSearch", &query, variables); err != nil {
    return fmt.Errorf("API call failed: %w", err)
  }

  if !query.Repository.HasDiscussionsEnabled {
    return fmt.Errorf("%s/%s has disabled discussions", repo.Owner(), repo.Name())
  }

  matches := []discussion{}

  for _, edge := range query.Repository.Discussions.Edges {
    searchText := edge.Node.Body + edge.Node.Title
    if strings.Contains(searchText, searchTerm) {
      matches = append(matches, edge.Node)
    }
  }

  if len(matches) == 0 {
    fmt.Fprintln(os.Stderr, "no matching discussions found :(")
    return nil
  }

  return nil
}
```

### Output formatting

We are querying the API but not yet actually printing anything out for the user. We could just print the results out raw, but what about concerns like:

- the width of the user's terminal?
- processing the output with a command like `cut`?
- visual pizazz?

`gh` frequently prints out a high density of information and relies on width-aware columnar printing that is friendly for both humans and scripts. All of the code that powers this is in `go-gh`, so let's make use of it in our extension.

First, add two new imports:

```go
import (
  // other imports
  "github.com/cli/go-gh/pkg/tableprinter"
  "github.com/cli/go-gh/pkg/term"
)
```

and then add new code to the bottom of `cli()`:

```go
package main
// import ...
// func main() ...
// type discussion...
// type gqlQuery...

func cli() {
  // input processing...
  // graphql call...

  isTerminal := term.IsTerminal(os.Stdout) // whether or not output is being piped to another program
  terminal := term.FromEnv()
  termWidth, _, _ := terminal.Size()

  tp := tableprinter.New(os.Stdout, isTerminal, termWidth)

  if isTerminal {
    fmt.Printf("searching for %s in %s/%s\n", searchTerm, repo.Owner(), repo.Name())
    fmt.Println()
  } // else don't bother printing if output is piped

  for _, d := range matches {
    tp.AddField(d.Title)
    tp.AddField(d.URL)
    tp.EndRow()
  }

  if err = tp.Render(); err != nil {
    return fmt.Errorf("couldn't print: %w", err)
  }

  return nil
}
```

Running our code now should return a result like:

```
go run . --repo cli/cli auth
searching for auth in cli/cli

Failing to authenticate on new wsl setup                                                                           https://github.com/cli/cli/discussions/6884
gh auth login on linux doesn't let me do git push (it asks for credentials)                                        https://github.com/cli/cli/discussions/6866
Cannot log in via browser or personal access token                                                                 https://github.com/cli/cli/discussions/6858
API call failed: USER does not have the correct permissions to execute `ClosePullRequest`                          https://github.com/cli/cli/discussions/6814

# rest truncated
```

Because we are using `go-gh`'s `tableprinter`, our command will behave appropriately if it gets piped. To see this in action, try:

```
go run . --repo cli/cli auth | cut -f1
Failing to authenticate on new wsl setup
gh auth login on linux doesn't let me do git push (it asks for credentials)
Cannot log in via browser or personal access token

# rest truncated
```

`cut` automatically splits input lines at a tab character; that's exactly what `tableprinter` uses as a delimiter when it detects it's being piped to another command.

## Publishing releases

`go-gh` can unlock a lot more functionality for our extension, but let's first pause on development to see how publishing an extension works.

Precompiled extensions like the one we're working on are distributed via binary assets on GitHub releases. Because we created our extension with `gh ext create`, our repository already has a GitHub actions workflow installed that looks like this:

```yaml
name: release
on:
  push:
    tags:
      - "v*"
permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: cli/gh-extension-precompile@v1
```

This workflow file means that when you push to a tag like `v1.0.0`:

- a new release gets added to your repository
- your Go code is compiled for you
- each target platform gets a binary uploaded as a release asset

These steps are performed by the GitHub Action [gh-extension-precompile](https://github.com/cli/gh-extension-precompile).

To see an example release, check out my [gh-ask repository](https://github.com/vilmibm/gh-ask/releases/latest).

It is the existence of a release like this -- complete with binary assets -- that `gh` looks for when you run `gh ext install`.

Go ahead and make a release!

```
git add .
git commit -m'it does things'
git push
git tag v1.0.0
git push origin v1.0.0
gh run watch
```

After the release is done, you can try installing and running your new extension:

```
gh ext install <your name>/gh-ask
gh ask --repo cli/cli auth
```

If you followed along, your extension repository is marked private and won't show up in commands like `gh ext search` and `gh ext browse`. Once you've published a release, all that is required to make it show up is to make your extension repository public.

## Other features in go-gh

At this point you have all the tools required to create and release a GitHub extension from end to end in Go.

There is much more to `go-gh`, however, so we'll take a tour through some of those extra features.

### Outputting JSON

We are attentive to whether or not our output is being piped to another program, but we can add an even deeper level of machine-readablity: JSON output.

`go-gh` helps us with two packages:

- `jsonpretty` which formats JSON for pretty printing
- `jq` which allows users to process the JSON we're outputting using [jq](https://stedolan.github.io/jq/) syntax

To take advantage of these, let's add some imports:

```go
import (
  // other imports
  "bytes"
  "encoding/json"
  "github.com/cli/go-gh/pkg/jq"
  "github.com/cli/go-gh/pkg/jsonpretty"
)
```

then add two new flags:

- `--json` for toggling JSON output
- `--jq` for processing JSON

```go
package main
// import ...
// func main() ...
// type discussion...
// type gqlQuery...

func cli() {
  jsonFlag := flag.Bool("json", false, "Output JSON")
  jqFlag := flag.String("jq", "", "Process JSON output with a jq expression")
  // rest of input processing...
  // graphql call...
  // output processing...
}
```

Flags in hand, let's add some new code right before we call `tableprinter.New()`:

```go
package main
// import ...
// func main() ...
// type discussion...
// type gqlQuery...

func cli() {
  // input processing...
  // graphql call...
  // isTerminal check...
  if *jsonFlag || *jqFlag != "" {
    output, err := json.Marshal(matches)
    if err != nil {
      return fmt.Errorf("could not serialize JSON: %w", err)
    }

    buff := bytes.NewBuffer(output)

    if *jqFlag != "" {
      return jq.Evaluate(buff, os.Stdout, *jqFlag)
    }

    return jsonpretty.Format(os.Stdout, buff, " ", isTerminal)
  }
  // rest of output processing...
}
```

Now if we run our code with `--json` we'll see:

```
go run . --repo cli/cli --json auth
[
 {
  "Title": "Failing to authenticate on new wsl setup",
  "url": "https://github.com/cli/cli/discussions/6884",
  "Body": "I'm in the process of setting up a new machine but in doing so I'm failing to authenticate with `gh auth login`.\r\n\r\nHere's the terminal output:\r\n\r\n```\r\n❯ gh auth login\r\n? What account do you want to log into? GitHub.com\r\n? What is your preferred protocol for Git operations? HTTPS\r\n? Authenticate Git with your GitHub credentials? Yes\r\n? How would you like to authenticate GitHub CLI? Login with a web browser\r\n\r\n! First copy your one-time code: <snip>\r\n- Press Enter to open github.com in your browser...\r\nfailed to authenticate via web browser: Too many requests have been made in the same timeframe. (slow_down)\r\n```\r\n\r\nIn the browser though the auth process goes through correctly, I entered the code, I approved access to my account/orgs, and I was presented with the success screen, only to find my terminal having the error.\r\n\r\nI appear to be on the latest release:\r\n\r\n```\r\n❯ gh version\r\ngh version 2.21.2 (2023-01-03)\r\nhttps://github.com/cli/cli/releases/tag/v2.21.2\r\n```\r\n\r\nAnd the WSL version is:\r\n\r\n```\r\n❯ lsb_release -a\r\nNo LSB modules are available.\r\nDistributor ID: Ubuntu\r\nDescription:    Ubuntu 22.04.1 LTS\r\nRelease:        22.04\r\nCodename:       jammy\r\n```"
 },
 ...
]

go run . --repo cli/cli --jq ".[]|.title" auth
Failing to authenticate on new wsl setup
gh auth login on linux doesn't let me do git push (it asks for credentials)
Cannot log in via browser or personal access token
# ... rest of output truncatedj
```

### Opening browsers

Sometimes you'll want your extension to open a a user's graphical browser. For example, in `gh` running `gh issue view 123 -w` opens issue 123 in a user's browser.

`go-gh` provides a function to help with this; it automatically figures out what browser to run and opens it for you.

To exercise this behavior in `gh-ask`, we'll add a new flag called `--lucky` which opens the first search result in a browser.

First, add a new import:

```go
import (
  // other imports
  "github.com/cli/go-gh/pkg/browser"
)
```

Then a new flag and some code to use it after our graphql call:

```go
package main
// import ...
// func main() ...
// type discussion...
// type gqlQuery...

func cli() {
  luckyFlag := flag.Bool("lucky", false, "Open the first matching result in a web browser")
  // other flags
  // graphql call
  if *luckyFlag {
    b := browser.New("", os.Stdout, os.Stderr)
    return b.Browse(matches[0].URL)
  }
  // rest of cli()
}
```

Now, running `go run . --repo cli/cli --lucky auth` should open https://github.com/cli/cli/discussions/6884 in your browser.

### Other features of go-gh

[Reference docs](https://pkg.go.dev/github.com/cli/go-gh#section-directories)

- Authentication parsing
  - As long as `gh` is logged in, your extension can take advantage of that auth
- REST API client
  ```go
  client, err := gh.RESTClient(&opts)
  if err != nil {
  	log.Fatal(err)
  }
  
  response := []struct{ Name string }{}
  if err = client.Get("repos/cli/cli/tags", &response); err != nil {
  
  fmt.Println(response)
  ```
- Markdown rendering
  - `fmt.Println(markdown.Render(pr.Body))`
- Configuration reading/writing
  - this can be used to read `gh`'s config and set values custom to your extension 
- Output templates
  - these can be exposed to users, allowing them to customize how your extension prints
- Text helpers
  - Pluralization, fuzzy time duration, rune width measurement

## Next steps

### Interpreted extensions

You don't have to precompile extensions; any executable file appropriately named in a repo can be installed as an extension. To learn more, check out the [official GitHub docs](https://docs.github.com/en/github-cli/github-cli/creating-github-cli-extensions#creating-an-interpreted-extension-with-gh-extension-create).

### Other precompiled languages

To work with a precompiled, non-Go language like C++ or Rust, we recommend adding a build script to your repo that can be called from the `gh-extension-precompile` action. To learn more, check out [these docs](https://github.com/cli/gh-extension-precompile#extensions-written-in-other-compiled-languages).

### Advanced Go CLIs

If you want to build a complex extension with multiple subcommands, check out [cobra](https://cobra.dev/). We use it in `gh`.

### Working with the CLI team

It is our hope that hubbers write more GitHub CLI extensions, especially to prototype new CLI features. If you'd like to work with us, check out our [Working With Us doc](https://github.com/cli/cli/blob/trunk/docs/working-with-us.md).

##  Wrapping up

Feel free to reach out to me on Slack or ask in the `#cli` channel if you have questions about `gh` or extensions!
