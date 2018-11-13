# Why do users have the district's clever ID as their clever ID?

## Problem

Sometimes districts who use clever have district admins or school admins who do not come over from their
Clever sync. Clever has only recently added the possibility of syncing district and school admins; not many customers
are using that functionality yet.

So administrators of clever districts in TTM need to have the ability to manually add users in TTM, much like
clever teachers can manually create classrooms and students in those classrooms that are not synced with clever.

But! We *only* allow adding of district or school admins if the current user is a clever user, as opposed to
non-clever-using admins who are allowed to add teachers as well. We also don't allow bulk import/invite of teachers
or students done by clever users.

This restriction is, as of the code versions linked to, currently implemented in:

* [invitation form decorator](https://github.com/thinkthroughmath/apangea/blob/514f474ebb72c1f0f21ce8b73cfa14b48b261bb7/app/decorators/invitation_form_decorator.rb#L7-L23)
* [the school admin user role](https://github.com/thinkthroughmath/apangea/blob/514f474ebb72c1f0f21ce8b73cfa14b48b261bb7/app/models/user_roles/school_admin.rb#L122-L124)

(yes these places have duplication that is ripe for refactoring)

Changing the logic based on whether the current user uses clever or not is also done in:

* [editing the districts' name](https://github.com/thinkthroughmath/apangea/blob/514f474ebb72c1f0f21ce8b73cfa14b48b261bb7/app/views/customer_districts/_form.html.haml#L18-L19), perhaps this should be checking `district.uses_clever?` though...
* [ability to create a new district](https://github.com/thinkthroughmath/apangea/blob/514f474ebb72c1f0f21ce8b73cfa14b48b261bb7/app/views/customer_districts/index.html.haml#L10-L12)
* [linking to bulk upload help docs](https://github.com/thinkthroughmath/apangea/blob/514f474ebb72c1f0f21ce8b73cfa14b48b261bb7/app/views/dashboard/_getting_started.html.haml#L29-L35) (since they're not allowed to bulk upload, they shouldn't see the help indicating that they can)
* [linking to the SIS status page](https://github.com/thinkthroughmath/apangea/blob/514f474ebb72c1f0f21ce8b73cfa14b48b261bb7/app/views/dashboard/quickstart/_district_admin.slim#L2-L3), we SHOULD show the link to district admins who use clever

* other places, git grep for `user.uses_clever?`

and restricts users' abilities based on the [presence of a clever_id value](https://github.com/thinkthroughmath/apangea/blob/514f474ebb72c1f0f21ce8b73cfa14b48b261bb7/app/models/user.rb#L376).

## Solution

We need to set these invited users' clever IDs to *something* so that we know they are part of a clever district and 
should not be allowed to add teachers.

We chose to use their district's clever ID since that will tie them to their district and be an identifyable value.

This is, as of the code versions linked to, implemented in:

* [customer_invitations controller](https://github.com/thinkthroughmath/apangea/blob/514f474ebb72c1f0f21ce8b73cfa14b48b261bb7/app/controllers/customer_invitations_controller.rb#L36-L38)
* [district_admins controller](https://github.com/thinkthroughmath/apangea/blob/514f474ebb72c1f0f21ce8b73cfa14b48b261bb7/app/controllers/district_admins_controller.rb#L16-L18)
* [school_admins controller](https://github.com/thinkthroughmath/apangea/blob/514f474ebb72c1f0f21ce8b73cfa14b48b261bb7/app/controllers/school_admins_controller.rb#L17-L19)

(yes these places have duplication that is ripe for refactoring)

## Other solutions

If you have other ideas on how to solve this problem that would provide advantages over this method, please propose 
them! :heart: This is certainly not the only way the problem could be solved, but it was what we settled on at the
time.
