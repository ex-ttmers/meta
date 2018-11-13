# New Initiative Checklist

In the past few years, we've done an excellent job of keeping administrative overhead to a minimum, while continuing to deliver features with excellence and quality. In that same time, we've found that there are times when lightweight project administration and some level of upfront discovery have helped us to stay organized and avoid potential pitfalls. 

The checklist below is meant be a _conversation starter_ for teams as initiatives are started, inclusive of both team management and technical considerations. _The checklist is not meant to be prescriptive._ Rather, it's meant to prompt discussion among team members. As all initiatives have a different level of complexity, some items on the checklist may not be relevant in all situations.

<hr>

✅ [What are roles and responsibilities across the initiative team?](#what-are-roles-and-responsibilities-across-the-initiative-team)

✅ [Should the initiative have a design or architecture review?](#should-the-initiative-have-a-design-or-architecture-review)

✅ [How will the initiative work interact with features or constraints already in place?](#how-will-the-initiative-work-interact-with-features-or-constraints-already-in-place)

✅ [Should the initiative have a slice plan?](#should-the-initiative-have-a-slice-plan)

✅ [Are there date expectations or date constraints for initiative completion that should be considered?](#are-there-date-expectations-or-date-constraints-for-initiative-completion-that-should-be-considered)

✅ [Should the initiative have a coordinated communications plan to internal stakeholders or customers?](#should-the-initiative-have-a-coordinated-communications-plan-to-internal-stakeholders-or-customers)

✅ [Should the initiative work be tracked in a single milestone or multiple milestones in Github?](#should-the-initiative-work-be-tracked-in-a-single-milestone-or-multiple-milestones-in-github)

✅ [Should the initiative work be viewed in rotation during the Monday standup?](#should-the-initiative-work-be-viewed-in-rotation-during-the-monday-standup)

✅ [Should the initiative work be planned using sprints?](#should-the-initiative-work-be-planned-using-sprints)

✅ [Should the initiative work be placed behind a feature flag?](#should-the-initiative-work-be-placed-behind-a-feature-flag)

✅ [Should user acceptance testing or customer pilot accompany the initiative?](#should-user-acceptance-testing-or-customer-pilot-accompany-the-initiative)

✅ [Should an in-application walkthrough accompany the initiative work?](#should-an-in-application-walkthrough-accompany-the-initiative-work)

✅ [Will the initiative result in an internal or customer facing change to reporting?](#will-the-initiative-result-in-an-internal-or-customer-facing-change-to-reporting)

✅ [Will the initiative result in changes to Customer Support tooling or the ability service customers?](#will-the-initiative-result-in-changes-to-customer-support-tooling-or-the-ability-service-customers)

✅ [Will the initiative result in changes to Content Team tooling or the ability to manage content?](#will-the-initiative-result-in-changes-to-content-team-tooling-or-the-ability-to-manage-content)

<hr>

### What are roles and responsibilities across the initiative team?
  * Should there be an 'initiative lead'?
    * If so, what kinds of things will this person be responsible for?
    * What are expectations for this person?
  * Who is considered the 'product owner' for the initiative?
    * What are expectations for this person?
    * Typically, the product owner and initiative lead should work very closely together.
  * Does the initiative require cross-team collaboration or coordination?
    * If so, who is responsible for facilitating?
  * How will direction or feedback from internal or external stakeholders be conveyed to the team and by whom?
  * What is the escalation path for a team member has an issue?
    * Technical and people issues may need different escalation paths.

<hr>

### Should the initiative have a design or architecture review?
  * If so, conduct synchronously with remote folks using GTM or traveling to the office (where possible), supported with asynchronous communication (where appropriate).
  * Ensure all appropriate voices are included in the review, which may vary by initiative.
    * Additional voices may include Systems Engineering, QA, CTO.
  * If design/architecture decisions change throughout the course of the initiative, ensure that this is communicated and re-reviewed (where necessary).
  
<hr>

### How will the initiative work interact with features or constraints already in place?
  * Benchmarks
  * Clever
  * Common Core (show vs. hide)
  * Content previews (item, activity)
  * Cross-state customers (CSUSA, Connections)
  * Customer archiving/school year rollover
  * Demo behavior (customers, teachers/admins, students)
  * District onboarding
  * Higher Ed 
  * IC dashboard
  * LACOE (special configuration for networking/static IP)  
  * Model home (aka 'mobile home', 'generate students')
  * Live Teaching '[Fast Pass](https://github.com/thinkthroughmath/apangea/blob/rc/lib/live_teaching_enrollment_item.rb#L20)'
  * Login service  
  * LTI
  * Motivation
  * Multi-classroom students
  * Pages (static, unauthenticated)
  * Parent portal
  * Technical debt (either as known constraint or potential to reduce/eliminate)

<hr>

### Should the initiative work be tracked in a single milestone or multiple milestones in Github?
  * Multiple milestones can be helpful for initiatives that have several large sub-parts.
    * This allows you to look at a separate Waffle board for each discrete sub-part or view all sub-parts combined into a single Waffle board.
  * Single milestones are usually all that are needed when an initiative is a series of small-ish tasks that need to be completed.
  * **Very** small projects or tasks (consisting of 1-2 Github issues max) can be worked in an existing milestone, such as Orkin.
  
<hr>

### Should the initiative have a slice plan?
  * In general, if the initiative is considered large or longer-running and is planning work using sprints, a slice plan should be created and maintained.
  * The slice plan should be regularly reviewed and updated as a team-owned artifact used to understand and communicate high-level status.
  * Slice plans are helpful for planning in user acceptance testing (UAT), QA reviews or incremental/iterative releases.
  * In most cases, the slice plan produced by the team will be 'translated' into an internal customer-facing document and placed on the Product Intranet, subject to Product's (Tim's and Heather's) discretion.
  * Slice plans take many forms and there is no one form that is right for every initiative.
  * Below are _examples_ of slice plans, but these are not the _only_ ways to construct a slice plan.
    * [Slice Plan: Multi-State Customers](https://github.com/thinkthroughmath/storyboard/issues/3681)
    * [Choose All That Apply Development/Release Plan](https://docs.google.com/a/thinkthroughmath.com/drawings/d/1b_gq-ajGd3_ck_pg6-T2LhW90wsLQUeT2aLNHYhGQxY/edit?usp=sharing)
    * [TTM Benchmark Development/Release Plan](https://docs.google.com/a/thinkthroughmath.com/drawings/d/1WubuZUMeYbCThHsFOfBWIdkv2IkzbOv5EmEmg2s6Zso/edit?usp=sharing)
    * [Avatar Builder Development/Release Plan](https://docs.google.com/a/thinkthroughmath.com/drawings/d/1-t21DLENmkVhNSdO6rFtjak2sdgKy1mjL0XxYGrQ7_0/edit?usp=sharing)
    * [Create/Evaluate Development/Release Plan](https://docs.google.com/a/thinkthroughmath.com/drawings/d/13hqNRw55kccD203WpxgkRq9lvvJxdslw-f7j7oqmS-Y/edit?usp=sharing)
    * [Weekly Progress Summary Reporting Development/Release Plan](https://docs.google.com/a/thinkthroughmath.com/drawings/d/15pCemApVbMjd7xEtF9Cnlm0S9vrP4bDtp8_zd3VazvQ/edit?usp=sharing)
    * [Reporting Definitions Development/Release Plan](https://docs.google.com/a/thinkthroughmath.com/drawings/d/1h6pKPGZjDdQI6DFzmWKELofn5ss8Tvqh7opRZC35zkM/edit?usp=sharing)
    * [Trollup Killa: Master Plan](https://github.com/thinkthroughmath/storyboard/issues/3530)
    * [Concept Zone Replacement Slice Plan](https://docs.google.com/a/thinkthroughmath.com/spreadsheets/d/1u_7uC5tipsS1verxuH4Eqo9IvPqXnaZnQ5LVo1_mWzg/edit?usp=sharing)
    * [Live Teaching Audio Slice Plan](https://docs.google.com/a/thinkthroughmath.com/spreadsheets/d/1lk6VzWUAUdzf0jRuN5jo_Z0O9yJgn1tsW7Oi1Ayv0yA/edit?usp=sharing)
    * [CleverQuest Slice Plan](https://github.com/thinkthroughmath/storyboard/issues/2230)

<hr>

### Are there date expectations or date constraints for initiative completion that should be considered?
  * Some initiatives may be requested to (ideally) be in place for a certain 'season' (i.e. fall onboarding, Spring test prep) or need to be 'gated' due to the nature of the changes (i.e. back-to-school or mid-year release).
  * If so, expectations should be discussed and considered as part of the slice plan, sprint planning and communications planning.
    
<hr>

### Should the initiative have a coordinated communications plan to internal stakeholders or customers?
  * Some initiatives, based on the scope of change or series of changes, should have a coordinated plan to communicate about development releases.
  * If needed, the plan should be coordinated between Engineering and Product (Tim and Heather).
  * Below are _examples_ of coordinated communications plans, but these are not the _only_ ways to construct a plan.
    * [Trollup Killa: Master Plan](https://github.com/thinkthroughmath/storyboard/issues/3530)
    
<hr>

### Should the initiative work be viewed in rotation during the Monday standup?
  * If so, coordinate with Steve to ensure the Waffle board(s) and filter(s) are set up appropriately for standup.
  * If the team is using multiple milestones to track work, determine whether to show the Waffle board for the current sub-part only or view all sub-parts combined into a single Waffle board.
  * Ensure an 'owner' of the board is designated.
    * The owner represents the board at the standup.
  
<hr>

### Should the initiative work be planned using sprints?
  * Sprint planning can be very helpful for large initiatives (in size, scope or impact) with large teams (generally ≥ 2 people), or for longer-running initiatives (≥ 1-2 months).
  * Even teams that decide not to plan work by sprints at the start of the initiative may find sprint planning useful later in order to drive the initiative to completion.
    * In some cases, teams have uncovered unexpected value in sprint planning from the start of an initiative.
  * If planning to use sprints, set up a regular series of planning/retrospective meetings at the end of each sprint.
     * Ensure all appropriate voices are included in the planning/retrospective meetings, which may vary by initiative.
       * Additional voices may include Systems Engineering, QA, CTO.
  * Ensure the Github sprint labels are used to plan and communicate work.
  
<hr>

### Should the initiative work be placed behind a feature flag?
  * It's generally a good idea to put the work behind a global feature flag (on/off for all customers), subject to Product's (Tim's and Heather's) discretion, based on the scope of the change and socialization required.
  * If planning to pilot a feature on a per-customer basis, a customer feature flag will be needed.
  * Identifying how features are intended to be rolled out can help to inform communication plans.
  
### Should user acceptance testing or customer pilot accompany the initiative?
  * For end-user facing changes (internal and customers), it's generally a good idea to allow end-users some time to review and absorb the changes and provide feedback.
  * If so, ensure the following (minimally) are clearly established:
    * Who is participating in UAT (Customer Support, Live Teaching, Content Team, ICs, customers).
    * Who is managing UAT from within the Product or Engineering teams.
    * Who is point of contact for UAT or pilot feedback.
  * UAT or customer pilots may require the need for [feature flags](#should-the-initiative-work-be-placed-behind-a-feature-flag).
  
<hr>

### Should an in-application walkthrough accompany the initiative work?
  * If so, ensure this work is planned-in (time is dedicated) and tracked in Github issues along with the rest of the work.
  * Like when using feature flags, identifying how features are intended to be rolled out can help to inform Product's (Tim's and Heather's) coordinated communication plans.

<hr>

### Will the initiative result in an internal or customer facing change to reporting?
  * If so, ensure Tim and Jim are included in order to socialize the change(s).
  * In most cases, an impact assessment (what will change, by how much, and for whom) is very useful for socializing/communicating changes.
  * Ensure that Periscope queries, views, dashboards and reports are considered in the impact assessment.
  
<hr>
  
### Will the initiative result in changes to Customer Support tooling or the ability service customers?
  * Ensure that the initiative doesn't impact current tooling or utilities.
  * Determine whether the initiative requires the need for **new** tooling or utilities.
  * Ensure that Customer Support is clear on how to speak to the initiative and troubleshoot any potential issues or pitfalls.
  * Ensure that Customer Support understands the level of escalation required if customers report issues based on the initiative.

<hr>
  
### Will the initiative result in changes to Content Team tooling or the ability to manage content?
  * Ensure that the initiative doesn't impact current tooling or utilities for maintaining content, such as the LMS/CMS.
  * Determine whether the initiative requires the need for **new** tooling or utilities.
  * New tooling or changes to existing tooling should be vetted with the Content Team to ensure adequate training and communication have  occurred.
