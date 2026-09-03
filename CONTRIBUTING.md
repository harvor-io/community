# Contributing to Harvor

Thanks for your interest in contributing to Harvor!

Harvor is an open-source project, and contributions of all kinds are welcome. Whether you're fixing a bug, proposing a feature, improving documentation, participating in a discussion, or helping another contributor, we appreciate your involvement.

## Before You Start

Harvor is currently under active development. APIs, architecture, project structure, and conventions may change as the project evolves.

For significant changes, new features, or architectural decisions, please start a discussion or issue before investing substantial time in an implementation. This gives maintainers and the community an opportunity to discuss the idea and make sure it aligns with the direction of the project.

Small bug fixes, documentation improvements, and other straightforward changes can generally go directly to a pull request.

## Finding Something to Work On

Check the Issues tab of the Harvor repository you'd like to contribute to.

Issues labeled `good first issue` are intended to be approachable for new contributors.

Issues labeled `help wanted` are areas where contributions would be particularly appreciated.

You can also participate in GitHub Discussions if you're looking for ideas or aren't sure where to start.

## Reporting Bugs

Before opening a bug report:

1. Search existing issues to make sure the problem hasn't already been reported.
2. Make sure you're using a reasonably current version of the project.
3. Gather enough information for someone else to reproduce the problem.

When reporting a bug, please include:

- A clear description of the problem
- Steps to reproduce it
- What you expected to happen
- What actually happened
- Relevant configuration
- Version information
- Logs or error messages when applicable

Please remove secrets, credentials, tokens, and other sensitive information before posting logs or configuration.

## Suggesting Features

Feature ideas are welcome.

For smaller, project-specific features, open an issue in the appropriate repository.

For larger ideas, new Harvor services, cross-project changes, or architectural proposals, start a GitHub Discussion first.

When proposing a feature, explain the problem you're trying to solve rather than only describing a particular implementation.

Understanding the underlying problem makes it easier to explore different solutions.

## Service Standards

[`FEATURES.md`](FEATURES.md) describes the cross-cutting features every Harvor
service strives to provide so that services are dependable, easy to use,
observable, and production ready (consistent REST conventions, idempotent writes,
OpenTelemetry, configuration as code, and so on).

When proposing or implementing a feature, or reviewing a pull request, use it as
a reference for the baseline expected of a Harvor service. It's a target to design
and review against rather than a gate every change must clear, and individual
repositories may add their own standards on top of it.

## Making a Contribution

The typical contribution workflow is:

1. Fork the repository.
2. Clone your fork locally.
3. Create a branch from the repository's default branch.
4. Make your changes.
5. Add or update tests when appropriate.
6. Run the project's tests and other validation locally.
7. Commit your changes.
8. Push the branch to your fork.
9. Open a pull request against the Harvor repository.

For example:

```bash
git clone https://github.com/YOUR_USERNAME/PROJECT.git
cd PROJECT

git checkout -b fix/descriptive-name

# Make your changes

git add .
git commit -m "fix: descriptive commit message"
git push origin fix/descriptive-name
```

Then open a pull request from your branch to the upstream repository.

Individual repositories may contain additional development, testing, or build instructions. Follow the README and contributing documentation in the repository you're modifying when those instructions differ from this general guide.

## Branches

Create focused branches for your work.

Descriptive branch names are preferred, for example:

```text
feat/add-oidc-provider
fix/token-expiration
docs/self-hosting
refactor/event-publisher
```

Avoid mixing unrelated changes into the same branch or pull request.

## Commits

Keep commits reasonably focused and use clear commit messages that describe the change.

Harvor generally follows the style of [Conventional Commits](https://www.conventionalcommits.org/):

```text
feat: add OIDC provider configuration
fix: handle expired access tokens
docs: add local development instructions
refactor: simplify event publishing
test: add authorization policy tests
chore: update dependencies
```

Don't worry about creating a perfect commit history while developing. Maintainers may squash commits when merging a pull request.

## Pull Requests

Pull requests should be focused and reasonably small whenever possible.

When opening a pull request:

- Clearly explain what changed.
- Explain why the change is needed.
- Reference related issues or discussions.
- Include tests when appropriate.
- Update documentation when behavior or configuration changes.
- Make sure existing tests pass.
- Respond to review feedback and questions.

Draft pull requests are welcome when you'd like early feedback on an implementation.

A pull request doesn't need to be perfect before starting a conversation.

## Code Quality

Contributions should follow the conventions already established by the project.

In general:

- Prefer simple solutions over unnecessary abstractions.
- Keep public APIs intentional and understandable.
- Handle errors explicitly.
- Avoid introducing dependencies without a clear benefit.
- Add tests for meaningful behavior.
- Document exported or externally visible functionality.
- Maintain backward compatibility when practical.

See [`FEATURES.md`](FEATURES.md) for the cross-cutting standards Harvor services
aim for. Project-specific standards may also be documented within individual
repositories.

## Documentation

Documentation contributions are just as valuable as code contributions.

This includes:

- Fixing unclear documentation
- Adding examples
- Improving setup instructions
- Documenting configuration
- Correcting typos
- Improving API documentation

If something was confusing while you were getting started, improving that documentation is an excellent contribution.

## Reviews

Code review is a collaborative process.

Maintainers may request changes before accepting a contribution. Feedback is intended to improve the project and maintain consistency across Harvor.

Contributors are also encouraged to review open pull requests and participate in technical discussions.

## Licensing

By contributing to a Harvor project, you agree that your contributions will be licensed under the license of the repository to which you are contributing.

Please do not contribute code that you do not have the right to submit.

## Security Issues

Please **do not publicly report security vulnerabilities through GitHub Issues or Discussions**.

Follow the security reporting instructions provided by the affected repository. If the repository has GitHub Private Vulnerability Reporting enabled, use that mechanism to privately report the issue.

## Getting Help

If you're unsure about something, ask.

Use GitHub Discussions for general questions, architectural conversations, contribution questions, and topics that span multiple Harvor projects.

For questions about a particular issue or pull request, use the conversation on that issue or pull request.

---

Thanks for helping build Harvor.

**Infrastructure you can trust.**
