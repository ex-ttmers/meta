# Opportunities for [Tenderloving](http://tenderlovemaking.com/) Care

It can often be a challenge to balance code improvements while
shipping features. However, managing significant change can often be a
challenge.

This document makes the process easier by documenting ongoing code
maintenance.

## Opportunities

- Upgrade Apangea to Rails 4.
  - Carol & Seejee have already done much of this, but there are tests
    that still need fixing.
- Profile tests & refactor things that are very slow.
- Convert {Slim,ERB} -> Haml
- Speed up specific slow tests, idenfified by jenkins. [See below](#slow-tests-in-jenkins).
- Simplify inheritance relationships in attempters.
- Move more code from application helper to hamburger helper.
- Move content from other wikis to this meta repository.
- Add database constraints or application level assertions and sanity checks where appropriate.
  For example, it would be nice to check for problems, such as is described in this commit:
  https://github.com/thinkthroughmath/apangea/commit/1a2c74a6e9d75bb76615d5eeae19157d53b983e4
- See also the tech debt trello board https://trello.com/b/nVHpeTgo/tech-debt



Feel free to add more.


## Hamburger Helper

At some point, rails started adding functions in ApplicationHelper to
*every* object by default. In order to remove this feature, a shim
called 'hamburger_helper' is introduce. The long term idea is:

1. Move all references to functions in application helper to hamburger
   helper. Replace calls to methods with the
   `hamburger_helper.<whatever method>` form.
2. At this stage, we now only have a *single* method that is being
   added to application helper, and that is hamburger_helper. So we've
   drastically reduced the size of the API exposed on each
   object. Additionally, all problematic calls are now prefixed by
   `hamburger_helper`.
3. Move methods that can be included elsewhere to their respective
   locations. For example, methods like
   `compressed_javascript_include_tag`
   could be included in application_controller and exposed with the
   `helper` method.
4. Once hamburger_helper is empty, delete it. At some point, also fix
   the 'application-helper-is-included-everywhere' setting.

## Carving up the Monorail

### Candidate Areas

Most of our application logic is in the single apangea Rails repo. Several pieces could be extracted and reimplemented separately to reduce the size of that repo:

* The activity feed is a prime candidate to be abstracted since the user experience requirements mimic well-established patterns from social media (but the code doesn't conform to them)
* Messages could be reimplemented (but have relatively low business value)
* We could pull authentication out in to a separate CAS-ish service. This would allow us to deploy a more sane shard strategy, where each customer's data could live on an entirely separate instance
* The Lesson Player could be expanded to handle all things Lesson Player related (or many).  One of the ideas was for the LP to handle attempts/serving/caching so that it was a stand-alone unit.  Whether the CMS was going to be included was discussed a bit, but not much.

### Areas of Improvement Prior to More SOA

We presently have 4 primary services that make up the Think Through Math LMS: Apangea (the LMS itself), Lesson Player, Live Teaching, and Data Warehouse.  As it stands we have some things we need to work on with regards to maintaining these services:

#### Deployments

We're getting better, but we need to better isolate our deployments between services.  Having to do a synchronized rollout is a pain overall, so when possible we tend to avoid that.  If one service has to be deployed before another in order for it to continue functioning, this is frowned upon.  If one servie has to be deployed upon in order for a new feature to become active, but not disrupting existing service, this is much more favorable than the former.

#### Versioning

Related to deployments, when we change our inter-service API's for any service, it should be done in a compatible way.  These can be handled much like we think of rolling deployments.  If you are adding fields to an API endpoint payload, this is okay because it will not disrupt a service's current operations.  If you modify the return values of an existing payload in an on backwards-compatible way, such as changing the return type of a key or completely removing a key, then this is considered a breaking change.  At this point, we should likely consider introducing some sort of concept of API versioning so that we do not need to synchronize deployments.  It will allow an existing service to continue chugging along happily until we complete the needed changes to allow it to take advantage of a new feature.

Since we do not share APIs with the general public, these versions do not need to be long lived if we do not want them to be.  As soon as all services have stopped utilizing a version we can get rid of it all together.

#### Synchronizing Feature Flags

We have had some concerns and some issues tied in with deployments of how to conceptually synchronize feature flags between applications.  In theory since we have independent services, synchronizing feature flags should not be a thing - they should function standalone.  However, with API versioning, we can effectively inform the other service how to perform, by allowing it to opt-in to using the new feature in a given situation.  Without versioning, we could also start allowing the services to pass a header back and forth of some sort to announce they want/have a given feature available while it's in a flagged state. Once the feature is in production we can simply remove the flag/header and make it a full feature.


## Slow tests in Jenkins

Slow tests are recorded by Jenkins. For a given branch (on e.g. http://jenkins.thinkthroughmath.com:8080/job/Apangea-rc/),
open the "last successful artifacts" listing.
You'll see "longTests-integration-1.txt", etc, and each split has this,
so there are like 20 of them and you can find these in each build for a test job
as well.
