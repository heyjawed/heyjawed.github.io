---
title:  "Git Commit messages - Best Practices"
author: "Jawed Salim"
date: 2022-12-20 13:04:58 +0600
description : "Git commit: How to write the better commit messages"
tags: [Git]
---

The `git commit` command captures a snapshot of the project's currently staged changes.
Committed snapshots can be thought of as *"safe"* versions of a project.
Git will never change them unless you explicitly ask it to.

Assuming you have already the basic understanding of [Git workflow](https://docs.github.com/en/get-started/using-git/about-git)

## Why you should write a better commit message?

You might say, **"It's just a personal project."** Yes, you work alone now, but what happens when you work with a team or contribute to open source?

Have you ever tried running `git log` on one of your old projects to see the **"weird"** commit messages you have used since its inception? It can be hard to understand **what** you have changed in past and **why** you made those changes in the past.
For example - `'Fix style' 6 months ago` # "yep....even you don't have the absolute idea about this commit"

Commit messages can adequately communicate why a change was made, and understanding that makes development and collaboration more efficient.

## The analysis of a Commit message

**Basic**

```sh
git commit -m <message>
```

**Detailed**

```sh
git commit -m <title> -m <description>
```

**Example**

```sh
git add README
git commit -m "Add Readme file"
```

**To write better commits, consider the following points:**

- Why have I made these changes?
- What effect have my changes made?
- Why was the change needed?
- What are the changes in reference to?

## How to write good commit messages

- **Specify the type of commit:**

| Commit Type | Title                    | Description                                                                                                 | Emoji  |
| ----------- | ------------------------ | ----------------------------------------------------------------------------------------------------------- |:------:|
| `feat`      | Features                 | A new feature                                                                                               | ✨     |
| `fix`       | Bug Fixes                | A bug Fix                                                                                                   | 🐛     |
| `docs`      | Documentation            | Documentation only changes                                                                                  | 📚     |
| `style`     | Styles                   | Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)      | 💎     |
| `refactor`  | Code Refactoring         | A code change that neither fixes a bug nor adds a feature                                                   | 📦     |
| `perf`      | Performance Improvements | A code change that improves performance                                                                     | 🚀     |
| `test`      | Tests                    | Adding missing tests or correcting existing tests                                                           | 🚨     |
| `build`     | Builds                   | Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)         | 🛠     |
| `ci`        | Continuous Integrations  | Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs) | ⚙️     |
| `chore`     | Chores                   | Other changes that don't modify src or test files                                                           | ♻️     |
| `revert`    | Reverts                  | Reverts a previous commit                                                                                   | 🗑     |

**Example** 

```sh
git commit -m "Fix (SRE-100): Bug preventing users from submitting the subscribe form"
```

- **Remove unnecessary punctuation marks**
- **Do not assume the reviewer understands what the original problem was, ensure you add it.**
- **Follow the commit convention defined by your team**

> The most important part of a commit message is that it should be clear and meaningful.

**Read more about coventional git commit messages [here](https://www.conventionalcommits.org/en/v1.0.0/)**

### Here are some examples of good and bad commits

**Good** 👍	
- Feat: Improve performance with lazy load implementation for images
- Chore: update npm dependency to latest version
- Fix: bug preventing users from submitting the subscribe form
- Update: incorrect client phone number within footer body per client request

**Bad** 👎
- fixed bug on landing page
- Changed style
- oops
- I think I fixed it this time?
- empty commit messages

## References

Below are the some excellent resources which help you to enhance your git knowledge and to become a professional "VCS":

- [https://docs.github.com/en/get-started/quickstart/set-up-git](https://docs.github.com/en/get-started/quickstart/set-up-git)
- [https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2)
- [https://learngitbranching.js.org/](https://learngitbranching.js.org/)
