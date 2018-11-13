# Intermittent Test Failures

This is a list of known intermittent failing tests, broken down by repo. Intermittent test failures make us lose trust
in our test suite, so we should try to fix them as soon as possible. If there is an intermittent test failure happening
often on a very active repo, such as Apangea, it is probably best to forego this list and either fix it immediately or open
an Orkin to fix it.  If the intermittent failure does not happen that often, is hard to reproduce, or exists on a repo that
currently isn't very active, then this list is a good place to put that intermittent test failure.  Feel free to add new
ones as they are discovered , or remove any that have been fixed. Please provide a stack trace and any other information
that may be helpful in debugging the failure. If it seems horribly out of date, or you have questions, Anthlam is the
person to yell at.

## Apangea

None

## Reporting

None

## Lesson Player

None

## Live Teaching

None

## Avatar Builder

1. the purchase confirmation when purchases have been made and save is clicked when the modal appears.the purchase confirmation when purchases have been made and save is clicked when the modal appears shows confirmation
   - Stack Trace:

     ```
     undefined method `[]' for nil:NilClass (NoMethodError)
     NoMethodError:
     undefined method `[]' for nil:NilClass
     ./spec/helpers/custom_selectors.rb:283:in `random_fixture_piece'
     ./spec/integration/purchase_confirmation_spec.rb:31:in `block (3 levels) in <top (required)>'
     ./spec/integration/purchase_confirmation_spec.rb:37:in `block (4 levels) in <top (required)>'
     ```

   - Additional Info:

2. points interaction.points interaction updates points after save attempt with insufficient funds
   - Stack Trace:

     ```
     Unable to find css ".points-piecesTotal" (Capybara::ElementNotFound)
     Capybara::ElementNotFound:
     Unable to find css ".points-piecesTotal"
     ./spec/helpers/custom_selectors.rb:62:in `block (2 levels) in <module:CustomSelectors>'
     ./spec/integration/points_spec.rb:68:in `block (3 levels) in <top (required)>'
     ```

   - Additional Info:

3. the purchase confirmation when purchases have been made and save is clicked when the modal appears.the purchase confirmation when purchases have been made and save is clicked when the modal appears lists unowned pieces on purchase confirmation
   - Stack Trace:

     ```
     undefined method `[]' for nil:NilClass (NoMethodError)
     NoMethodError:
     undefined method `[]' for nil:NilClass
     ./spec/helpers/custom_selectors.rb:283:in `random_fixture_piece'
     ./spec/integration/purchase_confirmation_spec.rb:31:in `block (3 levels) in <top (required)>'
     ./spec/integration/purchase_confirmation_spec.rb:37:in `block (4 levels) in <top (required)>'
     ```

   - Additional Info:

4. the purchase confirmation when purchases have been made and save is clicked when the modal appears.the purchase confirmation when purchases have been made and save is clicked when the modal appears totals the purchase price
   - Stack Trace:

     ```
     undefined method `[]' for nil:NilClass (NoMethodError)
     NoMethodError:
     undefined method `[]' for nil:NilClass
     ./spec/helpers/custom_selectors.rb:283:in `random_fixture_piece'
     ./spec/integration/purchase_confirmation_spec.rb:31:in `block (3 levels) in <top (required)>'
     ./spec/integration/purchase_confirmation_spec.rb:37:in `block (4 levels) in <top (required)>'
     ```

   - Additional Info:

5. points interaction.points interaction updates points when an unowned piece is worn
   - Stack Trace:

     ```
     Unable to find css ".points-piecesTotal" (Capybara::ElementNotFound)
     Capybara::ElementNotFound:
     Unable to find css ".points-piecesTotal"
     ./spec/helpers/custom_selectors.rb:62:in `block (2 levels) in <module:CustomSelectors>'
     ./spec/integration/points_spec.rb:30:in `block (3 levels) in <top (required)>'
     ```

   - Additional Info:

6. adjustments changing colors for a piece with multiple layers and a colorizable region.adjustments changing colors for a piece with multiple layers and a colorizable region clicking a palette color changes the colorizable color for each layer
  - Stack Trace:

    ```
    expected to find css "#pieces-jet-pack-wearing .pieces-colorizable" at least 1 time but there were no matches (Capybara::ExpectationNotMet)
    Capybara::ExpectationNotMet:
    expected to find css "#pieces-jet-pack-wearing .pieces-colorizable" at least 1 time but there were no matches
    ./spec/helpers/custom_selectors.rb:118:in `computed_style'
    ./spec/helpers/custom_selectors.rb:101:in `computed_colorizable_color_from_layer'
    ./spec/integration/adjustments_spec.rb:222:in `block (5 levels) in <top (required)>'
    ./spec/integration/adjustments_spec.rb:221:in `each'
    ./spec/integration/adjustments_spec.rb:221:in `block (4 levels) in <top (required)>'
    ```

## Login

None

## Post Deploy

1. a student.can connect to a live teacher
   - Stack Trace:

     ```
     test_0003_can connect to a live teacher(a student) [/usr/local/rvm/gems/ruby-2.2.4@apangea/gems/capybara-2.5.0/lib/capybara/node/finders.rb:43]:
     Capybara::ElementNotFound: Unable to find field "username"
     test/post_deploy/student_test.rb:119:in `block (3 levels) in <class:StudentTest>'
     test/post_deploy/student_test.rb:115:in `block (2 levels) in <class:StudentTest>'
     ```

   - Additional Info:
     - Possibly caused by farm data, but has failed on Dev and RC farms, and is a test that is doing a bunch of stuff with multiple sessions.
