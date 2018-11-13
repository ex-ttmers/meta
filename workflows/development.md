# Development Workflow

## Prerequisites

- [Git & GitHub](/onboarding/git.md)
- [Waffle](/services/waffle.md)

### (IMPORTANT) Nothing should be merged into rc that can't be shipped same day

## Assignment

Some project story (or Orkin duty) gets assigned to you (or you assign yourself if no tasks) and potentially your pair in GitHub.

## Preparing your environment

- Fetch the latest code for the repositories you'll be working with.
- Create and checkout a new feature branch off of `rc`.  This can be done with `git checkout -b branchname`
  - If you are working alone your branch name will be in the format `<initials>/<some-feature-name>`.  Ex. If Jane Doe wants to add Feature X, her branch name may look like `jd/adding-feature-x`.
  - If your are pairing with another developer your branch name will be in the format `<p1-initials>+<p2-initials>/<some-feature-name>`.  Ex. If Jane Doe and Stan Boar want to add Feature X, their branch name may look like `jd+sb/adding-feature-x`.
  - We are very loose, and often have fun, with our branch names so don't feel locked in to formal names, just make sure your initials prefix it.

## Writing the Code

Code your heart(s) out and have some fun adding new features or fixing existing ones in the code.  Be sure to ensure your code has proper test coverage and be cautious not to bite off too much for one feature.  We try to keep pull requests small so they are easier to review, so sometimes it's best to implement a story or a feature across multiple PRs so that they have isolated concerns and limited reach into other areas.

When making changes, try to make commits small and maintain their working order.  The idea is that if someone checks out a specific commit the app should still be able to run.  This is helpful to both you as the dev and others reviewing your code so that no changes get too overwhelmingly large or complex at one time.

Check your commits and comments for proper spelling and tolerable grammar so that your changes are easily understandable by all.

We have a Ruby [styleguide](https://github.com/thinkthroughmath/ruby-style-guide) that the team follows.  It's very long, so don't feel like you need to read through thoroughly.  We have Hound setup to review pull requests automatically and handle the linting for us.  It will make sure you know if something isn't up to par.

Our current styles for casing are as follows:
- Ruby code should always be snake case
- JSON should always be sent over HTTP with snake case keys
- JavaScript should always use camel case (converting JSON to camel case upon receiving and converting to snake case before sending)

## Submitting a Pull Request

Submitting a pull request (PR) should mark the end of a story, or at least a chunk of a story.  You are handing your code off to the rest of the team to review and approve.  This is one of the best parts of working on our team - constant feedback.  We strive for continual improvement of our own skills and our codebases, so getting more eyes on the code is a wonderful way to handle both.

When you create the pull request:

1. Give it a descriptive title.
  - Include the waffle story number (the GitHub Issue number on [Storyboard](https://waffle.io/thinkthroughmath/storyboard)) and optionally whether it fully fixes the story. Use [Waffle syntax](https://github.com/waffleio/waffle.io/wiki/FAQs#branch-moving) to connect the PR to the story. If you aren't sure, look at a done story on the current [Waffle board](https://waffle.io/thinkthroughmath/storyboard) to see how others have done it.
  - Refactors which simply restructure code should be noted as well: Ex. `[Refactor] Title of PR`

2. Add a full description, with what the PR accomplishes and acceptance criteria / how to test.
  - Describe what the PR accomplishes, important changes, and examples if appropriate
  - List any PRs that are blocking yours
  - List any PRs that yours is dependent on
  - Explain how QA should test your code
    - Write out thorough instructions so that QA can step through your code ensuring various things are as you expect
    - If you need to run specific rails console command like activating feature flags, be sure to list those in the appropriate order too
    - If you need to checkout certain branches on other services (lesson-player, live-teaching) in order to test your code, be sure to include this in the directions
  - List any special considerations that will be required when deploying your code in a section headed "Dependencies". Follow the format shown at https://github.com/TheJefe/gplan#dependency.
  - Spell check your PR to make sure it reads coherently

3. Label your PR
  - If you're still working on the PR and have pushed for tests or informal review, label it `WIP`.
  - If your PR is a child branch of another PR, label it `Child`. Child branches can't have `Needs Code Review` tags until their parent branch is merged and the child branch is rebased. Chains of nested PRs are difficult to understand and cause Code Review and QA churn as changes bubble up from their parent branches. If this is delaying your work, you can request a priority code review or add the `Blocking` label.
  - If you're done working and are ready for review, label it `Needs Code Review`
  - If your PR has a dependency that must be handled before merging, label it `Has Dependency`. Dependencies include:
    - PR is dependent on changes made in a PR on another repository.
    - Special actions must be taken to release the PR (e.g., another repo must be deployed in lockstep).
  - If your PR is blocking another PR or related work from moving forward, label it with `Blocking`.
  - If your PR requires downtime, label it with `Downtime Required`. For more info see: [What will require downtime?](../documentation/what-requires-downtime.md)
  - If your PR answers yes to any of the following questions, add the `Needs Release Notes` label:
      - 'Does this impact the end user?
      - 'Does this need any special communications to the field?
      - 'Do I not know if it does or not?'

To learn more about how the review process works, check out our [Pull Request Review](review.md) page
