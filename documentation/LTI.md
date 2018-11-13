## Learning Tools interoperability (LTI)

 https://en.wikipedia.org/wiki/Learning_Tools_Interoperability

 Support for LTI was added with this PR: https://github.com/thinkthroughmath/apangea/pull/2792

## Moodle

https://docs.google.com/a/thinkthroughmath.com/document/d/1sZmojiyiPgkd7bNUGqiIMgyjmvWXo1WIkkZ0WuG3kM4/edit?usp=sharing

## Canvas

https://docs.google.com/a/thinkthroughmath.com/document/d/14y_68iD2vjxAtL8aROBsHDMi7nRQW_lWvPNK_pcDGds/edit?usp=sharing

## LTI for Higher Ed

https://docs.google.com/document/d/1qSfjgxR2G2tq1ZHmPczGHveIB5ew4FTXX0yomGCcCf0/edit#heading=h.6e59gm5c11xr

## Testing LTI

Classlink offers a very simple [LTI testing tool](http://clweb.classlink.net/sandbox/), if you just want to see what will happen when different data is sent over LTI.

- set `Oauth Consumer Key` and `Oauth Consumer Secret` to match the values on the LTI district you are testing
- In this case, 'Resource link ID' is an identifier for the classroom, but is not currently used, so you can set it to anything.
- You'll need to add a field for 'custom\_school\_id' and it should equal the `external_id` of the TTM school.
- The launch URL will be like `http://localhost:5000/sso/auth/lti/callback`
- `Role` Should be `Learner` for a student, or `Instructor` for a teacher. It's rarely needed but you can set it to `Administrator` to create or match a school admin
- If you want to create a student in a specific grade level, you can add a field for 'custom\_grade\_level' and pass a number that matches an enrollable grade level
