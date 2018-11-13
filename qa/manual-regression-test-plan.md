# Apangea Manual Regression tests

### Purpose:

This document is intended as a suggested list of things to test for general major system/environmental updates, like ruby or rails upgrades.

**NOTE**: This is only a guideline, we are all smart people that can make smart decisions about what should be tested for a given change.

## Environmental

- [ ] Deploy apangea to a farm
- [ ] Run Performance tests
- [ ] Test across our [supported environments](https://content.thinkthroughmath.com/static/TTM-TechCheck.pdf)

## Accounts

- [ ] Create some TTM accounts, run ETL and verify they make it into the warehouse

## Student Experience

- [ ] Complete a Placement Test
- [ ] Complete a lesson, run ETL and verify it is recorded in the warehouse
- [ ] Build an avatar

## Teacher Experience

- [ ] Dashboard
- [ ] Benchmark Roster

## Reports

- [ ] Use the "Print" button on some reports
- [ ] Rollup reports
- [ ] Weekly Usage reports [meta](https://github.com/thinkthroughmath/meta/blob/master/documentation/mandrill.md#mandrill-test-all-the-things)
- [ ] Weekly Usage report emails get sent out
- [ ] Student progress report
- [ ] Overview Report
- [ ] Download Overview Report
- [ ] Standards Report
- [ ] Benchmark Growth Report
- [ ] Benchmark Performance Report
- [ ] Benchmark Administration Report

## Content

- [ ] Create an item
  - [ ] Upload an Image
- [ ] Publish an item
- [ ] Preview an item

## Live teaching

- [ ] Chat with a live teacher
- [ ] Live transcripts are sent over and can be viewed in apangea by a superuser

- - -

# Live Teaching Manual Regression tests

- [ ] Fast Pass works as expected. Regular students in a FastPass customer get picked up before other students
  - [ ] Demo Ready checkbox
  - [ ] "Demo" star indicator is shown
- [ ] Flash detection. Indicator should be shown, and initial message to student will not ask if the student would like to talk to text
- [ ] Spanish speaking indicator
- [ ] Higher Ed indicator
- [ ] New Student indicator
- [ ] Item information
- [ ] Chat Audio
- [ ] Report a student
- [ ] teacher stats tabs
- [ ] Admin Dashboard
- [ ] Chat Transcripts get sent to apangea
- [ ] Chat Reason is recorded
