## Adjusting student points

### Why would you adjust student points?
If we need to refund points to a student, or adjust the student's points for any other reason (they have too many or too few points), we can use the `PointCorrector` this in the rails console to accomplish this. For example:

```ruby
# Student 123 has
#   1,000 lifetime points
#     500 current points (meaning they've spent 500 over their time in the system)

# This means they have these point transactions:
# [1,000, 'earned points doing math']
# [-500, 'bought stuff']

# We removed a 250 point piece that we want to refund them for.

student = Student.find 123
PointCorrector.new( 
  student: student,
  current_points: 750,
  lifetime_points: 1000,
  reason: 'refunding for removed avatar piece'
).correct!

# This creates a new point transaction adding 250 points to the student:
# [250, 'refunding for removed avatar piece']
```

The intended use is for you to put in what you want the `current_points` and `lifetime_points` values to be, and it will do the math to get there. It doesn't work *quite* like that though; `lifetime_points` is a sum of __positive transactions only__ and `current_points` is a sum of __all transactions__. This means that at the end of the above example, student 123 will have 750 `current_points` and 1250 `lifetime_points`, even though you might have expected to only have 1000 `lifetime_points`. 

If we define lifetime points as all of the points ever earned over their lifetime, this actually fits the definition; they earned those points doing math and then earned them again when we refunded them. But it can be a little bit confusing.

Please note that in the example above you will get the same result if you set `lifetime_points` to 1250 instead of 1000.

If, for some reason points need to be taken away from a student, that works the same way:

```ruby

# student has 500 current points and 1000 lifetime points
PointCorrector.new( 
  student: student,
  current_points: 250,
  lifetime_points: 1000,
  reason: 'refunding for removed avatar piece'
).correct!

# Now student has 250 current points and 1000 lifetime points

```
