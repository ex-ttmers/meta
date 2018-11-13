# Waffle

We are using [waffle](https://waffle.io/) to track our user stories and issues. Waffle gives us another way to view our planned work and work in progress, by showing us GitHub issues and PRs in a kanban format. Here's the [waffle FAQ](https://github.com/waffleio/waffle.io/wiki/FAQs).

## Workflow with Waffle

[Here's](https://github.com/waffleio/waffle.io/wiki/Pull-Requests-&-Automatic-Work-Tracking) waffle's guide for pull requests and automagic work tracking.

### Creating an Issue

Issues are created in our [storyboard repository](https://github.com/thinkthroughmath/storyboard/issues). They are just regular GitHub issues. You can either create these issues in GitHub or directly on the [waffle board](https://waffle.io/thinkthroughmath/storyboard). Since a number of our issues are repo-spanning (or not obviously belonging to a particular repo on creation), we're going to keep all issues in the storyboard repo and reference them from our other working repos (apangea, reporting, lesson-player, etc).

We are organizing our issues using [milestones](https://github.com/thinkthroughmath/storyboard/milestones).

Once an issue is created, it will show up on the waffle board in the *Backlog* column.

### Issue Workflow

`Backlog` -> `WIP` -> `Needs Code Review` -> `Needs QA` -> `Closed`

In general, our issues should travel along the Waffle board from left to right. As an issue is being developed, it should be in the `WIP` column; moving to `Needs Code Review` once that is done. After two developers have reviewed the issue it should move to `Needs QA` and then on to `Closed` when the Issue is merged and done. If anything is raised to be addressed during the `Needs Code Review` or `Needs QA` phases, the Issue can move back into `WIP` and then continue back through each checkpoint from there.

Waffle uses Github Labels to determine what column the Issue lives in. Our Waffle board is set up to use the following:
  - __Backlog__: *No labels*
  - __WIP__: Waffle: WIP
  - __Needs Code Review__: Waffle: Needs Code Review
  - __Needs QA__: Waffle: Needs QA
  - __Closed__: *Not determined by labels; Issue is closed*

A problem with this, for us, is that we label our Issues based on Issue characteristics (eg: priority, type), and the associated PRs is where we like to keep track of work state (eg: wip, needs code review).

In order to keep our Issue labels in sync with associated PR labels, we have a [small label sync app](https://github.com/thinkthroughmath/GhLabelSync) running in Heroku, listening to [Github Webhooks](https://github.com/organizations/thinkthroughmath/settings/hooks/6486205) for our repositories. 

This means that you only have to change the PR labels to move an issue across the board. Here is an example scenario:

> __A PR is associated with an Issue using [Github or Waffle syntax](https://github.com/waffleio/waffle.io/wiki/FAQs#prs-and-issues).__ 
>   - Let's say we used the 'Closes' keyword. This means that when the PR is merged, Github will automatically close the Issue.
>   - Waffle will display the PR linked to the Issue on our board.

> __The `WIP`:tangerine: label is added to the PR.__
>   - The sync app will get the webhook notification and add `Waffle: WIP` to the associated Issue.
>   - The Issue will show, with associated PR, in the `WIP` column on our board.

> __The `Needs Code Review`:eggplant: label is added to the PR.__
>   - The sync app will get the webhook notification and add `Waffle: Needs Code Review` to the associated Issue.
>   - The Issue will show, with associated PR, in the `Needs Code Review` column on our board.
>   - Note that the Issue still has the `Waffle: WIP` label, but Waffle only shows it in whichever matching column is farthest right.

> __The `WIP`:tangerine: label is removed from the PR.__
>   - The sync app will remove the `Waffle: WIP` label from the Issue. 
>   - The board does not change.

> __The `Needs Code Review`:eggplant: label is removed from the PR, and the `Needs QA`:eggplant: label is added.__
>   - The sync app will change the Issue label to `Waffle: Needs QA`.
>   - The Issue will show up in the `Needs QA` column on our board.

> __The PR is merged.__
>   - The Issue will move to the `Closed` column on our board (but would not if 'connects' was used to associate the PR to the Issue).

> __The `Needs QA`:eggplant: label is removed from the PR.__
>   - The sync tool will remove the `Waffle: Needs QA` label from the associated Issue.
>   - There is no change to the board. 
>   - Note that at this point, the PR has no labels that would associate the Issue with a column, so if the Issue was not closed it would show up in `Backlog` instead.

This can seem more complicated with multiple PRs associated with an Issue, but in principle, it works the same. The Issue will reflect all labels from all associated PRs; but Waffle will only show it once on the board (the furthest right applicable column).

Issues can also be moved across the board by manually dragging them to a new column or changing their `Waffle: *` labels in Github.

>__Since the webhook only alerts the sync app on a label change, if you add a label to the PR and *then* associate it with the Issue, the sync app will not automatically update the Issue label. [There is an Orkin for this.](https://github.com/thinkthroughmath/storyboard/issues/2520) We could potentially fix this by doing a periodic full sync, but then we could not move any issues manually; they would revert to associated PR label state on sync.__

Issues only show in the *Done* column on the waffle board for 7 days.

## Waffle tips

- ⌘ + clicking on an issue in a waffle board will open the issue in GitHub (just clicking the issue will open up a card within the board)

## Issues with Waffle

Waffle is still a work in progress, and we've been running into some issues that they are working to address. They have their own waffle board of issues [here](https://waffle.io/waffleio/waffle.io).
