---
layout: post
title: "Internal design of Pronto"
date:   2017-03-15
categories: ruby
---

Pronto is ruby library which helps you do code review on your changes very
conveniently. It works with several linter, including rubocop, reek or brakeman.
It's not only output the issues in our terminal, but also putting comments PR,
which reduces a lot of time for us to review styling and convention.

Recently, #hardcore group of Vietnamrb intents to create a similar tool for
golang, and the first step for it is study how pronto works internally so that
we can make a golang version of it.

## Analyse

One of the most common command which we use is:

```
PRONTO_GITHUB_ACCESS_TOKEN=token pronto run -f github_pr -c origin/master
|                              | |        | |          | |              |
 ------------------------------   --------   ----------   --------------
            ||                      ||          ||              ||
            \/                      \/          \/              \/
        Access token           Main command    Formatter      Commit
     to use in Formatter     call Pronto::CLI
```

We'll take this command apart and examinate each of them.

- Firstly, we have `pronto run`, this is the main part of the command, which
will call `Pronto::CLI#run`, this is a class which inherits from [Thor](https://github.com/erikhuda/thor).
This CLI instance will take the options as arguments, then pass them to
`Pronto.run`. It also checks the current directory, and uses [Rugged::Repository](https://github.com/libgit2/rugged)
to get the current git workdir. Pronto will operate on the current git workdir.

- `-f github_pr` indicates a formatter, which helps the CLI generates an instance
of a subclass of `Pronto::Formatter::Base`, in this case, `Pronto::Formatter::GithubPullRequestFormatter`.
This instance is responsible for formatting the output of pronto checker, the
format can be standard output in terminal, or comments on github commits and
pull requests.

- `PRONTO_GITHUB_ACCESS_TOKEN=token` set an environment variable which the
formatter will use. Each formatter will use different variable.

- `-c origin/master` indicates a commit, and the CLI will pass this argument to
`Pronto.run`.

## Dive into the `Pronto.run`

![pronto run](https://lh4.googleusercontent.com/28o4wpdN0Rb4N2Kf9HhQohOAzXlZYX4iBQo1NS07rydqTsb_qM4f45uiQ9cKur43968VkXRv=w2100-h1270)

- `Pronto::CLI#run` will pass `repo_path`, `commit` and `formatters` to
`Pronto.run` as arguments.

- `repo_path` and `commit` is processed by `Pronto::Git::Repository` into
`patches`, a collection of `Pronto::Git::Patch` instances.

- The patch instances are inspected by `Pronto::Runners`, a collection of pronto
plugin to run the linter (ex: rubocop, reek or brakeman), then create result which
all formatters can read. The result is usually a collection of
`Pronto::Message`, each of which has the commit sha, file path, line number of
the error, severity level of the error and the runner name.

- The `formatters` then read the result and create the expected output.

## How `Pronto::Runners` work, then?

- `Pronto::Runners.runners` gets all the subclasses of `Pronto::Runner::Base`,
  this behaviour is defined in module `Pronto::Plugin`. By default, a
  `Pronto::Runners` instance will have all runners to inspect the patches.

- Each runner will run separately through all the valid changes, each time the
runner encounters an offence, if that offence is not disabled, runner'll save it into result.

- Take `pronto-rubocop` for example, firstly, it uses `Rubocop::ProcessedSource`
to process the whole file and get the offences, then it checks
if the offence line number is included in the added lines. After getting all the
offences in the added lines, it creates the result with path, line number and
severity.

## Conclusion

- The idea of pronto is pretty simple, combines with the elegant implementation,
  which makes it very convenient for me to study the codebase.

- From the note above, I'll need to check the golang ecosystem to see if
we can benefit from them like pronto does:

1. Linters which has good architecture that separates processing and output
formatting, so that we can modify the output easily. I saw that there are
[golint](https://github.com/golang/lint), [govet](https://golang.org/cmd/vet/) and [metalinter](https://github.com/alecthomas/gometalinter) which are very promising.
2. Git client similar to rugged so that we can manipulate git repository to get
all the patches and changes. Lib2git already published a git client similar to
rugged in golang: [git2go](https://github.com/libgit2/git2go)
