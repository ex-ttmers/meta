### Constants

A number of the points and multipliers are global to the application,
but they're defined in a YAML file that's loaded at application
startup. The file is `config/general_constants.yml`. We'll make
references to the values in that file in this document, but also
provide their values so it's not abstract.

### Bonus points

* Passed lesson: 250
* Completed placement test: 250 (note: AFAICT these are not stored
  anywhere in the TTM app)

### Lesson multiplier

If you've done this particular lesson previously you'll get fewer points:

* 1st time taking lesson: *1*
* 2nd time taking lesson (aka "RETAKE"): *0.5* (config: `second_attempt_multiplier`)
* 3rd time taking lesson (aka "DEFERRED"): *0.25* (config: `third_attempt_multiplier`)

Note that this multiplier only applies to points on individual
questions, *not* on bonus points.

Additionally, we *always round up() when we don't get whole
numbers. So if a student gets the first attempt correct on a multiple
choice question the second time through a lesson (multiplier 0.5)
she'll get 13 points, not 12.5. And the third time through the lesson
(multiplier 0.25) she'll get 7, not 6.25.

### Testing out bonus points

* Passing Pre-Quiz on the lesson's 1st attempt: 500 (config: `tested_out_lesson_bonus_points`)
* Passing Pre-Quiz on the lesson's 2nd attempt: 250
* Passing Pre-Quiz on the lesson's 3rd attempt: 0

Note that these bonus points are rewarded on top of the 250 bonus points for
passing the lesson.

### Attemptable types

We currently have two attemptable types in the system:

* ItemStep => points as listed below
* Chapter => 0 points

This may change as a result of next generation math content, stay
tuned.

### Multiple choice and Fill in the blank

Activities that allow multiple attempts:

* 1st attempt correct: 25  (config: `mc_1st_try_points`)
* 2nd attempt correct: 10  (config: `mc_2nd_try_points`)
* 3rd attempt correct: 5   (config: `mc_3rd_try_points`)
* 4th attempt correct: 1   (config: `mc_4th_try_points`)

Activities that do not allow multiple attempts (pre- and post-quiz):

* 1st attempt correct: 25  (config: `mc_1st_try_points`)
* Item failed: 0

### Problem solving process (PSP) - New Lesson Player
* Points awarded after completion of _entire_ PSP (subject to lesson multipliers): 50

### Examples

#### 1. Lesson: Pre-quiz test out

Pre-quiz activity has 7 questions and does not allow multiple tries
per question:

    Q1. CORRECT   - 25 * 1x points
    Q2. CORRECT   - 25 * 1x points
    Q3. INCORRECT -  0 * 1x points
    Q4. CORRECT   - 25 * 1x points
    Q5. CORRECT   - 25 * 1x points
    Q6. CORRECT   - 25 * 1x points
    Q7. CORRECT   - 25 * 1x points

    ACTIVITY TOTAL: 150

Show your work!

      150 = 25 points * 6 attempts

#### Lesson 1: Totals

Since this pathway is marked as lessons being skippable if a valid
pre-quiz score achieved, this lesson is completed. What were the total
points for the lesson?

Earned points:

    Pre-quiz:        150
    ====================
    LESSON ACTIVITY: 150
       Passed bonus: 250
       LESSON TOTAL: 400

Possible points:

    Pre-quiz:        175
    ====================
    LESSON ACTIVITY: 175
       Passed bonus: 250
       LESSON TOTAL: 425

#### 2. Lesson: Five activities

##### 2.1: Pre-quiz non test out

Pre-quiz activity has 7 questions and does not allow multiple tries
per question:

    Q1. CORRECT   - 25 * 1x points
    Q2. INCORRECT -  0 * 1x points
    Q3. INCORRECT -  0 * 1x points
    Q4. INCORRECT -  0 * 1x points
    Q5. CORRECT   - 25 * 1x points
    Q6. CORRECT   - 25 * 1x points
    Q7. CORRECT   - 25 * 1x points

    ACTIVITY TOTAL: 100

Show your work!

      100 = 25 points * 4 attempts
    =====
      100

##### 2.2: Guided learning

Guided learning activity has 5 questions and allows multiple tries per
question:

    Q1. CORRECT   - 25 * 1x points
    Q2. INCORRECT -  0 * 1x points
        INCORRECT -  0 * 1x points
        CORRECT   -  5 * 1x points
    Q3. INCORRECT -  0 * 1x points
        CORRECT   - 10 * 1x points
    Q4. CORRECT   - 25 * 1x points
    Q5. INCORRECT -  0 * 1x points
        INCORRECT -  0 * 1x points
        INCORRECT -  0 * 1x points
        CORRECT   -  1 * 1x point

    ACTIVITY TOTAL: 66

Show your work:

      Q1. 25
      Q2.  5
      Q3. 10
      Q4. 25
    + Q5.  1
    ========
          66

##### 2.3: Focus

Focus activity has 3 chapters, and attempts are recorded to note that
the student has viewed the chapter (~flash movie):

    Q1. Viewed - 0 points
    Q2. Viewed - 0 points
    Q3. Viewed - 0 points

    ACTIVITY TOTAL: 0

##### 2.4: Practice

Practice activity has 8 questions, and allows multiple tries per question.

    Q1. INCORRECT -  0 * 1x points
        INCORRECT -  0 * 1x points
        CORRECT   -  5 * 1x points
    Q2. INCORRECT -  0 * 1x points
        INCORRECT -  0 * 1x points
        CORRECT   -  5 * 1x points
    Q3. INCORRECT -  0 * 1x points
        CORRECT   - 10 * 1x points
    Q4. CORRECT   - 25 * 1x points
    Q5. INCORRECT -  0 * 1x points
        INCORRECT -  0 * 1x points
        INCORRECT -  0 * 1x points
        CORRECT   -  1 * 1x point
    Q6. INCORRECT -  0 * 1x points
        INCORRECT -  0 * 1x points
        CORRECT   -  5 * 1x points
    Q7. INCORRECT -  0 * 1x points
        CORRECT   - 10 * 1x points
    Q8. CORRECT   - 25 * 1x points

    ACTIVITY TOTAL: 86

Show your work:

      Q1.  5
      Q2.  5
      Q3. 10
      Q4. 25
      Q5.  1
      Q6.  5
      Q7. 10
    + Q8. 25
    ========
          86

##### 2.5: Post-quiz

Post-quiz activity has 7 questions, and does not allow multiple tries
per question.

    Q1. CORRECT   - 25 * 1x points
    Q2. CORRECT   - 25 * 1x points
    Q3. INCORRECT -  0 * 1x points
    Q4. INCORRECT -  0 * 1x points
    Q5. CORRECT   - 25 * 1x points
    Q6. CORRECT   - 25 * 1x points
    Q7. CORRECT   - 25 * 1x points

    ACTIVITY TOTAL: 125

Show your work!

      125 = 25 points * 5 attempts
    =====
      125

##### Lesson 2: Totals

What were the totals for the lesson?

Earned points:

    Pre-quiz:        100
    Focus:             0
    Guided learning:  66
    Practice:         86
    Post-quiz:       125
    ====================
    LESSON ACTIVITY: 377
       Passed bonus: 250
       LESSON TOTAL: 627

Possible points:

    Pre-quiz:        175 (7*25)
    Focus:             0 (Chapters only)
    Guided learning: 125 (5*25)
    Practice:        200 (8*25)
    Post-quiz:       175 (7*25)
    ====================
    LESSON ACTIVITY: 675
       Passed bonus: 250
       LESSON TOTAL: 925

#### 3. Lesson: Retake of five activities including PSP

This is a RETAKE of the original lesson.

##### 3.1: Pre-quiz non test out

Pre-quiz activity has 7 questions and does not allow multiple tries
per question:

    Q1. CORRECT   - 25 * 0.5x points = 13
    Q2. INCORRECT -  0 * 0.5x points
    Q3. INCORRECT -  0 * 0.5x points
    Q4. INCORRECT -  0 * 0.5x points
    Q5. CORRECT   - 25 * 0.5x points = 13
    Q6. CORRECT   - 25 * 0.5x points = 13
    Q7. CORRECT   - 25 * 0.5x points = 13

    ACTIVITY TOTAL: 52

Show your work!

       52 = 13 points * 4 attempts
      ===
       52

##### 3.2: Guided learning

Guided learning activity has 5 questions and allows multiple tries per
question:

    Q1. CORRECT   - 25 * 0.5x points = 13
    Q2. INCORRECT -  0 * 0.5x points
        INCORRECT -  0 * 0.5x points
        CORRECT   -  5 * 0.5x points =  3
    Q3. INCORRECT -  0 * 0.5x points
        CORRECT   - 10 * 0.5x points =  5
    Q4. CORRECT   - 25 * 0.5x points = 13
    Q5. INCORRECT -  0 * 0.5x points
        INCORRECT -  0 * 0.5x points
        INCORRECT -  0 * 0.5x points
        CORRECT   -  1 * 0.5x point  =  1

    ACTIVITY TOTAL: 35

Show your work:

      Q1. 13
      Q2.  3
      Q3.  5
      Q4. 13
    + Q5.  1
    ========
          35

##### 3.3: Problem solving

    ACTIVITY TOTAL: 50 * 0.5x points = 25

##### 3.4: Practice

Practice activity has 8 questions, and allows multiple tries per question.

    Q1. INCORRECT -  0 * 0.5x points
        INCORRECT -  0 * 0.5x points
        CORRECT   -  5 * 0.5x points =  3
    Q2. INCORRECT -  0 * 0.5x points
        INCORRECT -  0 * 0.5x points
        CORRECT   -  5 * 0.5x points =  3
    Q3. INCORRECT -  0 * 0.5x points
        CORRECT   - 10 * 0.5x points =  5
    Q4. CORRECT   - 25 * 0.5x points = 13
    Q5. INCORRECT -  0 * 0.5x points
        INCORRECT -  0 * 0.5x points
        INCORRECT -  0 * 0.5x points
        CORRECT   -  1 * 0.5x point  =  1
    Q6. INCORRECT -  0 * 0.5x points
        INCORRECT -  0 * 0.5x points
        CORRECT   -  5 * 0.5x points =  3
    Q7. INCORRECT -  0 * 0.5x points
        CORRECT   - 10 * 0.5x points =  5
    Q8. CORRECT   - 25 * 0.5x points = 13

    ACTIVITY TOTAL: 46

Show your work:

      Q1.  3
      Q2.  3
      Q3.  5
      Q4. 13
      Q5.  1
      Q6.  3
      Q7.  5
    + Q8. 13
    ========
          46

##### 3.5: Post-quiz

Post-quiz activity has 7 questions, and does not allow multiple tries
per question.

    Q1. CORRECT   - 25 * 0.5x points = 13
    Q2. CORRECT   - 25 * 0.5x points = 13
    Q3. INCORRECT -  0 * 0.5x points
    Q4. INCORRECT -  0 * 0.5x points
    Q5. CORRECT   - 25 * 0.5x points = 13
    Q6. CORRECT   - 25 * 0.5x points = 13
    Q7. CORRECT   - 25 * 0.5x points = 13

    ACTIVITY TOTAL: 65

Show your work!

       65 = 13 points * 5 attempts
    =====
       65

##### Lesson 3: Totals

What were the totals for the lesson?

Earned points:

    Pre-quiz:         52
    Guided learning:  35
    Problem solving:  25
    Practice:         46
    Post-quiz:        65
    ====================
    LESSON ACTIVITY: 223
       Passed bonus: 250
       LESSON TOTAL: 473

Possible points:

    Pre-quiz:        175 (7*25)
    Guided learning: 125 (5*25)
    Problem solving:  50
    Practice:        200 (8*25)
    Post-quiz:       175 (7*25)
    ====================
    LESSON ACTIVITY: 725
       Passed bonus: 250
       LESSON TOTAL: 975

## Problems

__1. Non-final attempts shouldn't have any score at all.__
__2. Incorrect attempts shouldn't have any score at all.__

This hasn't been a huge problem to now because the DW enters a 0 for
`earned_points` on attempts that are neither final nor correct, so the
incorrect points in that respect are confined to apangea. But if we're
going to roll them up it would be better to not have the filtering
logic in multiple places.

__3. Placement test bonus points are not stored anywhere___

Are these in the DW? Not sure.

If we wanted to store them on apangea we should do so on the
Enrollment, but then that complicates scoring aggregation so we might
push that method to the enrollment and have it do something like:

    def earned_points
      lesson_enrollments.sum(:earned_points) + placement_test_points
    end

    def placement_test_points
      if placement_test_completed?
        APP_CONFIG['placement_test_points'] # << doesn't exist yet!
      else
        0
    end
