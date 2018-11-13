# Report and Data requests

As TTM becomes more and more heavily reliant on data to drive our business, we want to make sure that we operationalize the production, support and quality of the data and reports we generate to answer business questions. This document describes the process for submitting the report, working with stakeholders, generating and reviewing the report, and operationalizing the data.

## Prerequisites

- [Git & GitHub](/onboarding/git.md), specifically the [data-scripts](https://github.com/thinkthroughmath/data-scripts) repo
- [waffle](https://waffle.io/thinkthroughmath/storyboard). This document makes heavy reference to the [Waffle workflow](/services/waffle.md). Any direction here for how to use waffle is for convenience only - the waffle workflow page is the canonical source.


## Requests & Assignment

A stakeholder that wants a report should use the [form] to make the request. We do this so that we can track the volume of reports over time so we can make sure we have appropriate staffing, and also so that we can identify trends in requests that are opportunities to automate. The result of the request form is a ticket added to waffle with a [specific milestone](https://waffle.io/thinkthroughmath/storyboard?milestone=Report%2FData%20Requests). Tickets are initially assigned to Jim, who will work with the project managers to assign resources to handle the report requests.

## Building the report

It's up to the implementer to create tools and/or queries needed to produce the report. Where possible, please point all queries to a non-production data source, such as the read-only followers on the apangea databases or in the case of the data warehouse, [Periscope](https://periscope.io) which heavily caches the warehouse data.

Any code or queries should end up in the [data-scripts](https://github.com/thinkthroughmath/data-scripts) repo so they can be re-used if needed. Similar to [development workflow](/workflows/development.md), you should create a branch in this repo and store your changes there. 

It's perfectly acceptable and encouraged to work with the stakeholder to iterate through the report to make sure what you are building matches their expectations, especially where there may be ambiguity in the nature of the request.

## Submitting a Pull Request

As with the other development processes, in order to have an additional set of eyes on the work we use a code review process. From the [data-scripts](https://github.com/thinkthroughmath/data-scripts) site, after you push your branch generate a Pull Request to the `master` branch. Use the [Waffle syntax](https://github.com/waffleio/waffle.io/wiki/FAQs#branch-moving) to connect your PR to the associated request in Waffle.

When you create the pull request:

1. If it makes sense (and in most cases it couldn't hurt, copy the report request data over. 
2. Include the contact info for the stakeholder in case the person doing the code review needs to contact them.
3. Include the database on which the report should be run. If it's designed to work on Periscope (and if it is, the query is probably also implemented there), indicate as such and link to the dashboard where it's implemented.

After you submit the pull request move the card in Waffle to Working (branch made) and add the 'Needs Code Review' label.
