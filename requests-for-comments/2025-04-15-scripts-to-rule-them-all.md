# Request for comments: Consistent scripts to rule them all

## Problem statement

*Who are we building for, what are their needs (include example cases), why is this important?*

PostHog has 235 (at the time of writing) public repositories. Each of these repositories has a unique way to get the project up and running locally.
Each repository has a unique way to run tests, build the project, and so on. For those of us who work on the many client SDK libraries, this adds a
lot of friction as we switch from project to project and have to learn a whole new set of incantations. Not only that, it makes it more challenging
for external contributors to get involved as they have to learn a whole new process for each repository they work on.

And as we continue to grow, this challenge will only grow.

## Solution

Adopt and adapt GitHub's approach of [scripts to rule them all](https://github.blog/engineering/engineering-principles/scripts-to-rule-them-all/) to
fit our needs.

> Being able to get from git clone to an up-and-running project in a development environment is imperative for fast, reliable contributions.

I'm not suggesting we do _exactly_ what they do, but a similar approach would be a big benefit. Not every repository will need every script. Some
repositories will need scripts custom to the environment. That's all fine. The goal is to have a baseline set of scripts that we can use to get a
project up and running in a development environment and a known location to look for those scripts.

## Context

For context, here's the list of scripts referenced in the GitHub blog post ([samples available in a repository](https://github.com/github/scripts-to-rule-them-all)):

* `script/bootstrap` - Install dependencies (Ex. npm packages, brew packages, etc. Usually run once after cloning the repository)
* `script/setup` - Setup the project. Seems like this could just be merged in bootstrap
* `script/update` - Update project after a pull such as migrations, etc.
* `script/server` - For web apps, starts the application. For client SDKs, might start an example site.
* `script/test` - Run tests (Ex. `npm test`, `bundle exec rspec`, etc.). Also includes linting, formatting, etc.
* `script/cibuild` - used by the CI system to run tests, build, etc.
* `script/console` - is used to open a console for your application.

## Design

For `posthog/posthog`, we already have a set of scripts in the `bin` directory. One issue with using `bin` across the board is that some environments
add `bin` to the `.gitignore` file by default because that's where they compile binaries to. However, we've already set this precedence, so it may make
sense to stick to it. For example, [posthog/posthog-dotnet](https://github.com/PostHog/posthog-dotnet/tree/main/bin) already has a `bin` directory.

* `bin/setup` - Install or upgrade dependencies (Ex. npm packages, brew packages, etc. Usually run once after cloning the repository).
* `bin/update` - Updates dependencies after a pull. This could simply call `bin/setup`.
* `bin/build` - Build the project, for projects that are compiled such as C#, Java, etc.
* `bin/start` - Start the project. For SDKs, this might start an example server.
* `bin/test` - Run tests (Ex. `npm test`, `bundle exec rspec`, etc.). Also includes linting, formatting, etc.
* `bin/fmt` - Optional: Format/lint code. This can be called by `test`.

## Alternative Ideas Considered

I've noticed a lot of our repos have a `make` file. This is especially common for mobile development. An alternate proposal would be to standardize on `make` rather than these `bin` scripts.

My recommendation is to still standardize on `bin` scripts so that it's consistent across the board. Some of the bin scripts are pretty complicated and it'll be hard to port to `make`.
If some repos have `make` files, that's fine. The bin scripts can call the `make` commands.

## Feedback Requested

* What other scripts do we need?
* What should the scripts be named?
* Should we use `bin` or something else?
* Should we also define repository templates for new repositories?

## Next Steps

I'll leave this open for two weeks to gather feedback.

After that, if there's consensus on the approach:

1. I'll create a PR to the Handbook with documentation on our approach.
2. I'll create a scripts repository that contains some template scripts (and script utilities).
3. I'll create a PR to each SDK repository to add the new scripts as I go.
