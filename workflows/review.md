# Pull Request Review Process

All those labels you added while you were
[working on code](development.md) are so the rest of the team knows
what state your pull request is.  Since our codebase is quite
extensive we've settled on a process that allows us to ensure a
certain amount of integrity with each PR.  We walk through these
states in a given order: `Code Review -> QA Review`.

## Code Review

Each pull request currently requires two developers to review the code
in a PR.  This helps us keep the code up-to-par, help catch any
potential performance or regression issues, prevent accidental bugs if
a deprecated API method is used, and help each other find better ways
to approach problems.

Our code review process usually flows linearly:

### Assignment

1. mathbot, Steve Robbibaro, or yourself will assign you and another
   developer a PR.
1. You and the other developer can work alone or pair on the pull
   request

### Reviewing

This is more than a human linting/compiling process.  Code reviews
should preferably be fairly thorough.

Some questions to ask yourself as you review:


#### High-Level Questions
- Why does this PR exist?
- What does this PR accomplish?
- In what way does it accomplish it?
- Is the PR description sufficient? Ex:
  - Is the overall description acceptable?
  - Are the QA steps good?
- Are the tests passing?

#### Code-Level Questions

##### Principles of Good Code
- Is there any redundant or duplicate code?
- Is the code as modular as possible?
- Can any global variables be replaced?
- Could this have been implemented by any well-supported third-party solution? The best code is code we don't write.

##### Clarity & Cleanliness
- Is all the code easily understood?
- Are the commits clearly organized?
- Is anything unclear about the code? Ex:
  - Class, method, or symbol names
  - Communication of intent
  - Anything that is surprising
- Does it conform to your agreed coding conventions? Hound will check for many of these, but a human review is always helpful
- Is there any commented out code?
- Can any logging or debugging code be removed?
- Is there any code elsewhere in the app that should change because
  - this PR means that it is now dead?
  - this PR introduces duplication on the code level or on the idea level?

##### Correctness
- Does the code work? Does it perform its intended function?
- Are there any flaws in the Authors logic? Ex:
  - Reversed booleans
  - Bad selection of objects
  - Mismatched types
- Does each commit work individually? (Are they "atomic")
- Is there sufficient test coverage for these changes?
- Are there any performance issues in this code, such as
  - DB calls inside of loops?
  - heavy work inside loops?
  - heavy re-calculations that could be memoized?

##### Deployability
- Have any agreed-upon feature flags been included and used properly?
- Are there database migrations that would require downtime? See ["What will require downtime"](https://github.com/thinkthroughmath/meta/blob/master/documentation/what-requires-downtime.md#what-will-require-downtime).
- Are there any considerations that may impact a production environment?
- Does this require release notes or special communication to the field?
    - If not, add the `Passed Release Notes` label.
    - If yes, add the `Needs Release Notes` label.
    - If unsure, add the `Needs Release Notes` label.

Raise any concerns to the developer as comments and line comments on
GitHub. Developers should think about these things as they are
writing the PR.

If the pr answers yes to the question 'Does this impact end users?' (including bug fixes) then you need to add the label `Needs Release Notes` and write a comment telling the pr owner to communicate with Tim to discuss what needs to be done before this pr can get merged and deployed.

If you approve the code then leave a comment stating it's good, remove
the `Needs Code Review` label and add `Passed Code Review` and `Needs
QA` flag.

If you do *not* approve the code in its current state then remove the
`Needs Code Review` label and add `Needs Work` along with all
appropriate comments.

## QA Review

Once Code Review has been completed, QA will begin.  The QA team
workflow is as follows:

- Work out of GitHub Repo Issues
  - Sort by label `Needs QA`
  - or use [DevWizard](https://dev-wizard.herokuapp.com/needs_qa)
  - or use [gi-web](https://github.com/anthlam/gi-web) or just [gi](https://github.com/TheJefe/gi)
  - Also take into account any team key priorities that might have been brought up at standup

- After selecting an issue..
  - QA based on issue's description acceptance criteria
  - If passed AND latest build passes, then merge it into `rc`
  - If failed
      - Leave comment as to why it was not passed
      - Remove `Needs QA` label
      - Label as `WIP`
  
If the pr answers yes to the question 'Does this impact end users?' (including bug fixes) then you need to add the label `Needs Release Notes` and write a comment telling the pr owner to communicate with Tim to discuss what needs to be done before this pr can get merged and deployed.

**Do not Merge until the PR has passed code review, QA, and release note labels.**

## Recovering from Needs Work

If your PR was labeled as `Needs Work`, it means it failed at one of
the three reviews and you may have a bit more work to do.  If it
failed at code review, look over the comments.  Sometimes the reviewer
simply marks as `Needs Work` until you have addressed all their
questions and they may not require a change to be made. If this is the
case, the reviewer will change the label to `Passed Code Review` and
add the `Needs QA` label.

If you failed a code review and there are things you need to address,
make those changes, change the label back to `Needs Code Review` and
wait for re-review.  This cycle may need to be repeated a few times,
but usually just once will do it.

If you failed QA and it wasn't a real failure, QA will change the
label to `Passed QA` upon completion and merge in your code.  If you
failed QA for a valid reason then you'll likely need to change code
and will need to reapply the `Needs Code Review` and `Needs QA`
labels.

## Priority Reviews

Sometimes you *need* to get a pull request through the pipeline fast.
In these situations request that someone review it in our Engineering
Hipchat room and/or ask Steve Robbibaro to assign someone.  After that
be ready to make all needed changes quickly so that you don't become
the bottleneck.
