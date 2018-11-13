# Model Home / Generate Students

### Q: What is a "model home", and how do I generate one?

"Model home" refers to a feature to add some realistic data (e.g. students and student activity) to a classroom for preview purposes. Fortunately, this works well for us developers too!

Model home data is added on a per-classroom basis. It may only be added to customers that are "demo".

In order to generate model home data for a classroom

1. Be logged in as an administrator.
1. Go to a classroom page (e.g. http://localhost:5000/schools/1/classrooms/1). If you do not have a classroom, create one. This also applies to Classroom-required records (e.g. Schools and Customers).
2. Click the "Generate Students" button on the classroom page.

### Q: What if we need to change the data being used for the "model home"?

The data for the model home is pulled from a real classroom, and then the names are scrubbed. If we need to update the model home data (because it's showing data from the previous school year, or lessons that are retired, or for some other reason), we can ask Sales to select a representative classroom that they like and run something like the following in the rails console:

```ruby
c = Classroom.find 263248
file_name = '263248.json'
archive = File.open file_name, 'w'
serializer = ClassroomDataSerializer.new c
serializer.extract_to archive
archive.close
```

You should then have a json file you can download. You can replace the `app/assets/javascripts/classroom_snapshots/generated_students_data.json`, and you will have to update the number of students in `SchoolClassroomsController.generate_students` to match the number of students in your classroom. All of this could be automated, but it isn't yet.

Additionally, you may want to adjust the date in `ClassroomDataLoaderWorker` if you run 'Generate Students' with your new data and your dates seem out of whack (benchmarks finished in the future, that sort of thing).

If for some reason, Generate Students isn't working after you've pulled a new file, you can test just the import in your console this way:

```ruby
path = '263248.json'

user = User.find 6            # seeded teacher
classroom = Classroom.find 1  # demo classroom

file = File.new(path)
loader = ClassroomDataLoader.new(classroom)
loader.load(user, file)
```

An example of a PR that updates the generated students data is [apangea PR 3203](https://github.com/thinkthroughmath/apangea/pull/3203).
