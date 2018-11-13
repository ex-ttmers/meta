# Think Through Math Engineering Processes

## Introduction

This repository serves as documentation for TTM engineering processes and knowledge.

We are using this to record existing, and collaboratively add or change, processes.

## Primary Nav

* data - Mostly random files that were in the apangea wiki that probably don't have a use anymore but looked potentially important.
* documentation - Development-related knowledge.
  * data warehouse - the repo and database we use for storing reporting data.
  * how-to - instructions for things you are likely to do while developing (as opposed to on the live site, which is under operations)
  * sso - single sign on.
* ideas - Things that haven't been implemented yet but might be in the future.
* images - If any images are referenced throughout these pages, they'll be in here.
* misc - Stuff that doesn't fit anywhere else; not development related.
* onboarding - Knowledge a new employee is most likely to need first.
* operations - Documentation, processes, and instructions having to do with the production site.
* services - Third party services, code, or tools that we use.
* techdebt - Information about improvements we'd like to make to our code or infrastructure.
* troubleshooting - Problems people have had and (hopefully) instructions on how to fix them.
* workflows - More process-y stuff about how we do things.

## Objective

At the time of creation we largely have our processes defined within the wiki on the apangea repository on GitHub.  This has served us pretty well
for housing information, but did not aid us in changing processes.  They were commonly changed in ways that were hard to track or discuss.

Here we can submit an issue to propose discussion on a topic, and submit pull requests to fully propose a change to practices.  This way all engineers
will receive alerts about a new discussion topic and are free to discuss it until agreements are reached.  This does not rule out having formal meetings
on topics at all, but discussion points in the meeting should be documented on the appropriate pull request or issue thread.

## Getting Started

### File Name Format

Submit any new markdown pages with a slugified, lowercase, hyphen-case name and directory in order to conform to a "standard" for the repo which will also relate to how they are mapped with HTML anchors.  This is proposed to match up with HTML attribute style and how URL styles are generally favored.

#### Examples:
- My Super Awesome Page => my-super-awesome-page
- My ! overly & punctual (Super Awesome Page) => my-overly-punctual-super-awesome-page
- My_Underscored Page => my-underscored-page
- My----VERBOSE Page => my-verbose-page

### Opening Discussions or Proposing Changes to Processes

Check out the [meta documentation](https://github.com/thinkthroughmath/meta/blob/master/workflows/meta.md) to see our workflow for discussing and proposing changes to internal processes.

## History

Many of these pages used to be in the github wiki associated with the apangea repo, but that stopped making sense for a variety of reasons.

If you find a page where the links are broken, that's probably why.

If you find a page where the images are broken, the images probably exist in the `/images/` directory in this repo but they're not linked properly.
