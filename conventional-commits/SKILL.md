---
name: write-commit-message
description: Use when creating git commits, writing commit messages, or when about to run git commit -- enforces structured commit message format with imperative mood
metadata:
  version: "1.0"
---

# Overview

All commit messages should follow a consistent format. This document outlines their structure and guidelines for writing them based on conventional commits.

# Format
All commits should be in the format

```
<type>: <description>

[optional body]

[optional footer(s)]
```

## Types
the following are allowed values for the 'type'

- **fix** - patches a bug in your codebase (this correlates with PATCH in Semantic Versioning).
- **feat** - introduces a new feature to the codebase (this correlates with MINOR in Semantic Versioning).
- **chore** - a change to project build files or configuration that does not introduce new user functionality
- **docs** - a change to project documentation without changing application functionality
- **test** - a change to introduce or modify testing code without changing application functionality.

A commit that  introduces a breaking API change (correlating with MAJOR in Semantic Versioning) has a footer `BREAKING CHANGE:`

footers other than BREAKING CHANGE: <description> may be provided and follow a convention similar to git trailer format.

## Scope
Never include a scope in the commit message.

# Examples
## Commit message with description and breaking change footer
```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

## Commit message with no body
```
docs: correct spelling of CHANGELOG
```

## Commit message with multi-paragraph body and multiple footers
```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: #123
```

## Commit message with both ! and BREAKING CHANGE footer
```
feat!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

# Specification
1. Commits MUST be prefixed with a type, which consists of a noun from the 'type' section.
2. The type feat MUST be used when a commit adds a new feature to your application or library.
3. The type fix MUST be used when a commit represents a bug fix for your application.
4. A scope should never be provided
5. A description MUST immediately follow the colon and space after the type prefix. The description is a short summary of the code changes written in the imperative mood, e.g., fix: array parsing issue when multiple spaces were contained in string.
6. A longer commit body MAY be provided after the short description, providing additional contextual information about the code changes. The body MUST begin one blank line after the description. 
7. A commit body is free-form and MAY consist of any number of newline separated paragraphs.
8. One or more footers MAY be provided one blank line after the body. Each footer MUST consist of a word token, followed by :<space> separator, followed by a string value (this is inspired by the git trailer convention).
9. A footer’s token MUST use - in place of whitespace characters, e.g., Acked-by (this helps differentiate the footer section from a multi-paragraph body). An exception is made for BREAKING CHANGE, which MAY also be used as a token.
10. A footer’s value MAY contain spaces and newlines, and parsing MUST terminate when the next valid footer token/separator pair is observed.
11. Breaking changes MUST be indicated in as an entry in the footer.
12. If included as a footer, a breaking change MUST consist of the uppercase text BREAKING CHANGE, followed by a colon, space, and description, e.g., BREAKING CHANGE: environment variables now take precedence over config files.
13. If included in the type prefix, breaking changes MUST be indicated by a ! immediately before the :. If ! is used, BREAKING CHANGE: MAY be omitted from the footer section, and the commit description SHALL be used to describe the breaking change.
14. A commit type and description MUST be written in lower case.

# FAQ

## What do I do if the commit conforms to more than one of the commit types?
Go back and make multiple commits whenever possible. Part of the benefit of Conventional Commits is its ability to drive us to make more organized commits and PRs.

## How does this relate to SemVer?
fix type commits should be translated to PATCH releases. feat type commits should be translated to MINOR releases. Commits with BREAKING CHANGE in the commits, regardless of type, should be translated to MAJOR releases.


