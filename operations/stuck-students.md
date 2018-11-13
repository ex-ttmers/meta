Note: This feature is still WIP, see Carol or Jeff if you have suggestions.

This info is from the testing instructions in [PR #1508]

This is an attempt at giving us a way to find out about students who are stuck due to a problem with our code (as opposed to students who are getting out of sync errors because they're trying to cheat). I'm imagining engineering as the primary users of this page, but it would be accessible to support as well.

The idea is that if a student gets a successful attempt after they get an out of sync error, they're likely cheating. But if they get multiple out of sync errors and have not been able to make an attempt since then, it's more likely that they're stuck and we should take a look at it.

To test:

- [ ] Preview a pre-quiz activity and immediately open 3 tabs to the same problem.
- [ ] In another browser session, as a superuser, go to /stuck_students.
- [ ] Attempt the problem in the first tab; this should work fine and if you reload /stuck_students, there should still be 0 for all types.
- [ ] Attempt the problem in the second tab; you should get an out of sync error. If you reload /stuck_students, your student should appear in the "Potentially stuck students" table.
- [ ] Attempt the problem in the third tab; you should get an out of sync error again. If you reload /stuck_students, your student should appear in the "Stuck students" table.
- [ ] Go back to the first tab and attempt the NEXT problem like normal; this should NOT get an out of sync error. If you reload /stuck_students, your student should appear in the "Unstuck students" table.
