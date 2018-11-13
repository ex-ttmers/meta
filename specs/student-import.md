# Student Import

<!-- MarkdownTOC -->

- [Manual Add](#manual-add)
  - [Project Goals & Assumptions](#project-goals--assumptions)
  - [Slice Plan](#slice-plan)
  - [UI Design](#ui-design)
    - [Guidelines](#guidelines)
    - [Initial Mockups for Manual Add](#initial-mockups-for-manual-add)
    - [Workflow](#workflow)
    - [Adding Students](#adding-students)
      - [How does adding students work currently?](#how-does-adding-students-work-currently)
      - [Student management proposal](#student-management-proposal)

<!-- /MarkdownTOC -->

<a name="manual-add"></a>
## Manual Add

<a name="project-goals--assumptions"></a>
### Project Goals & Assumptions

This portion of the student import project is intended as a way for teachers who are less tech-savvy to be able to bulk enter students. We do hope that in general, this is a last resort, and that students are added via district onboarding, clever, or a bulk import; but given that those things won't happen for a teacher (or haven't happened yet), we still need a way for teachers to get their kids into the system and doing math as quickly as possible.

We spoke to some of the teachers in TTM about their process, and it sounds like we are dealing with a huge range of tech skill, as well as a range of ways that teachers get their roster. Ideally we would want teachers to be able to copy/paste students in, but it sounds like (especially for the less tech-savvy) we can't count on that as an option, since a lot of "Digital Gradebooks" don't have an easy way to just get student names or basic info. Also, since we don't know where they are coming from, if we wanted to try to support it we would have to make the ability to accept pasted data as general as possible, and building / supporting that is out of the scope of this project. 

Some teachers understand CSVs and a bit of data manipulation - the CSV import is probably a better choice for those. The manual add is meant for the teachers that don't want to do that (and from our understanding wind up having to type the list of students somewhere anyway). To keep it a little bit less overwhelming we are intending for teachers to add students within the context of a single classroom at a time, and breaking up the data entry into specific chunks.

<a name="slice-plan"></a>
### Slice Plan
- [x] **Slice 1 - Proof of Concept and UI Work**
  - [x] Elm Proof of Concept
  - [x] Collect Data in POC
    - [x] Add / Select Classroom
    - [x] Enter Student Information
    - [x] Enter Student Credentials
    - [x] Enter Student Additional Info
    - [x] Delete Student 
  - [x] Add Progress Bar to top
  
- [x] **Slice 2 - Make It Real**
  - [x] Validate Data *(In Progress)*
    - [x] Show validation errors on student info
    - [x] Show validation errors on student credentials
    - [x] Show validation errors on extra student info
    - [x] Show validation errors on classroom
    - [x] Prevent forward nav if errors
    - [x] Make grade level a drop down
  - [x] Display Results & Approve/Edit
  - [x] Support roles (Admin/Teacher)
  - [x] Save User State (so users leave and come back) 
    - [x] Nav
    - [x] Store & reconstitute in-progress data
  
- [ ] **Slice 3 - Big Picture and How Import Fits into TTM Overall**
  - [ ] Integrate TurboTax version with csv Import
    - [x] Link to existing csv import
    - [ ] *Nice to have:* Move csv import code into elm and away from jquery steps
  - [ ] Search/Add Students

<a name="ui-design"></a>
### UI Design

<a name="guidelines"></a>
#### Guidelines

- Think Turbo-Tax, or "walkthrough"
- We should keep screens as atomic and reasonably simple as possible; only asking one question at a time.
- This interface should be conversational and friendly so we don't scare off non-technical users.
- The "wizard" idea isn't really a "steps" idea where you are meant to be able to navigate through steps; rather, it is meant to railroad a user into following a process where they don't have to think. Allowing forward / back navigation adds another user choice, and there isn't a lot of benefit.
- However, there may be a few explicit instances where we allow the user to navigate back to a specific step (notably in the review section, to fix or update information).
- We do show an indication of "steps" completed only so the user has an idea of how far they are along in the process (and hopefully feels like there aren't a lot of steps so this is "easy").
- If they don't want to finish an import they can just leave.
- But we did talk about saving state - if they are spending a lot of time entering this information, we don't want them to lose it. That means we'll have to store import state per user, so we can jump back to wherever they were in the process.
- On the tech side - we are planning to use this wizard to "build up" a set of data that will run through our existing sync process.

<a name="initial-mockups-for-manual-add"></a>
#### Initial Mockups for Manual Add

These were created in Sketch and are not meant to be "pixel-perfect" or have final copy - but meant to give the team an overall idea of the finished product and as a basis for ongoing conversation and iteration.

| Screen |
|--------|
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Welcome.png?raw=true) |
| Welcome Screen |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%201.png?raw=true) |
| Classroom Setup |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%202a.png?raw=true) |
| Classroom Setup - Choose an Existing Classroom |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%202b.png?raw=true) |
| Classorom Setup - Create a Classroom |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%203a.png?raw=true) |
| Classroom Setup - Auto Enrollment for an Existing Classroom |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%203b.png?raw=true) |
| Classroom Setup - Auto Enrollment for a New Classroom |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%204.1.png?raw=true) |
| Student Add (need to enter required fields) |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%204.2.png?raw=true) |
| Student Add (add a new student) |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%204.4.png?raw=true) |
| Student Add |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%204.5.png?raw=true) |
| Student Add (error) Side note - I think this dialog should 'grow' rather than getting a scroll bar, they're always going to be entering students at the end so the forward navigation won't get lost and it keeps it simpler for the less tech-savvy. Also keeps it simple for us (all we need to worry about is a min-height). |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%204.6.png?raw=true) |
| Student Add (error when editing a previous row) |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%205.png?raw=true) |
| Student Credentials |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%206.png?raw=true) |
| Student Credentials |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%207.png?raw=true) |
| Additional Student Info |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%208.png?raw=true) |
| Additional Student Info |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%209.png?raw=true) |
| Review Updates (would include classroom assignments) |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%2010.png?raw=true) |
| Review Additions |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%2011.png?raw=true) |
| System is working... (this may seem silly as a step on its own, but I do like that we have it in the 'progress bar' as a way of letting them know that they aren't stuck if they make a mistake) |
| ![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/Manual%20Import%2012.png?raw=true) |
| All done, with summary |

<a name="workflow"></a>
#### Workflow

![](https://github.com/thinkthroughmath/meta/blob/master/specs/assets/student-import/manual-add/manual-add-flow.png?raw=true)

<a name="adding-students"></a>
#### Adding Students

<a name="how-does-adding-students-work-currently"></a>
##### How does adding students work currently?
###### As a teacher

Currently, a teacher can add a single student by going to the classroom list and clicking `Add Students` for a particular classroom, or viewing a classroom roster and selecting `Add Students` there. This brings you to a page that has a form for a single student on the top, and a section on the bottom to search for and add existing students by Student Information Number. These actions can only occur in a classroom context.

There are links to import students in the Quick Start Menu, at the top of the Student List, at the top of the Add a Student page, and on the Classroom Roster. 

Entity management is not done through the top menu. The entrie  s under `Students` are currently `Student List` and `Benchmark Roster`.

<a name="student-management-proposal"></a>
##### Student management proposal

On the top level student menu, add a new link titled `Add Students`. This link will bring you to the `Welcome` page of TurboDumbleDalf.

The initial mocks list two options:
  - I want to add students to TTM by importing a file.
    - This option links to the current CSV import.
  - I want to manually add students to TTM.
    - This walks the user through adding a number of students in a classroom context.

There are 2 additional modes of entry that we want to cover:
  - Adding just one student at a time 
    - This could potentially be done with the existing manual add; does it even make sense to build a new form that is streamlined for adding just one student at a time? If we add a different way to do it, we add another branch in the dialog that the user has to navigate.
    - If we do want a new form, should the option be up front on the initial welcome screen, or should we ask how many students are to be added after we establish the classroom context?
  - Searching for and adding existing students
    - This could be a new option on the welcome page: "I would like to add existing students to a classroom."
    - Choosing this option should walk you through the same screens as the normal manual add up to classroom selection.
    - After classroom selection, there would be a new screen that allows you to enter any number of SINs. Currently, they can be pasted into a box separated by commas or line breaks. A good first pass would be to continue processing these the same way.
    - On the next screen, you are presented with a list of any unmatched SINs and the option to go back and edit the SINs (this is skipped if all of the students are found).
    - Next, you are presented with a list of all of the matching students (similar to how it currently works). You can select which students from that list you would like to add to the classroom that you selected earlier.
    - Finally, a summary of the associations that were made is displayed.

