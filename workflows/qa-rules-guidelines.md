### QA is just another Software Developer Skill

The best Quality Assurance Engineers can read and write code.  As is true that a good developer can test code.  This has already been proven through TDD.  Just like some devs are more skilled in front end or backend parts of the app, so is true with QA. QA is just another developer skill.

## Recommended testing process

1. Read the story and ask questions if there's not enough information on the ticket about what the behavior used to be, what the behavior should be after the change, what parts of the app the change might influence, what types of users this might affect, etc.
2. Check to make sure that the PR has passed automated tests on Jenkins.  We can make exceptions for known intermittent failures
3. git the latest RC
4. merge in the PR branch locally, or on a hosted staging farm
5. validate the feature works as intended based on the story
6. If the change affects the UI, check in all the browsers we support.
7. check that it doesn't break any other parts of the app that it may influence.
8. Does it need to be performance tested? Push the branch to a farm and run [this job](http://jenkins.thinkthroughmath.com:8080/job/performance-test/)
8. Accept the story, merge the pull request and delete the branch.
10. File new issues for anything you discover along the way that isn't correct but wasn't broken by the change you're testing.

## QA Rules

1. You are never allowed to "accept" a story that you developed on.
2. You should validate on your own system or one of the hosted farms
3. Anything visual should be checked in Chrome, Safari, Firefox, IE9, IE10, iPad, iPad mini, and an Android tablet.  Note that we will punt on some mobile administration issues.



## #DevsDoingQA

- Joel has found this link very helpful for deciding where to jump in to do QA: https://github.com/issues?q=is%3Aopen+is%3Apr+user%3Athinkthroughmath+label%3A%22Needs+QA%22+sort%3Aupdated-asc
