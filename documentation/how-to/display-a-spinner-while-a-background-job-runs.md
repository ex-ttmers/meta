# Display a spinner while a background job runs

Occasionally, part of a request requires work to occur in the
background. A common pattern in the application is to present the user
with a spinner while the background job completes.

The jobs controller and abstraction provides an easy way to implement
this pattern in your own code.

## What is a Job?

A Job, for the purposes of this discussion, is a 1-1 mapping between
Sidekiq jobs and our own Job concept, along with Job metadata. It
provides infrastructure around job completion/lifecycle. Jobs are
created/looked up according to the sidekiq job id -- it provides a
convenient way to specify the job. Job metadata is stored in Redis as
JSON, similar to how Sidekiq does it.

As an aside, you might be wondering why we cannot use the data that is
already in sidekiq to to the same thing instead of storing it
separately. The trouble is, sidekiq deletes job metadata when the job
completes, so the argument data is unavailable for post-completion
handling. It would be awesome if completed jobs' metadata was
persisted, and this would be a good 10% time activity.

## Creating a new Job

A Job can be created with the JobRepository:

```ruby
repo = JobRepository.new

repo.create(:id => job_id, ...)
```
The options for creating a job are:

1. `:id` -- the ID to associate with the job. This could be anything, but
   it should be unique. The sidekiq job ids are very convenient for
   this.

2. `:handler_class` -- A class to handle actions around jobs. This should
   be the string class name (which would also include any enclosing
   modules, e.g. "FooController::MyHandlerClass").

3. Any additional metadata/hash params. These will be JSON serialized
   and will later be available via the `data` method on a `Job`.


### Create Jobs inside Sidekiq Worker Class Method

Since `Job`s and workers are closely related, it makes sense to create
jobs inside class methods. For example:


```ruby
class MyNiceClassroomWorker

  include Sidekiq::Worker

  def perform(classroom_id)
    repo = JobRepository.new
    job = repo.find(self.jid)

    #
    # do something nice here
    #

    job.data[:was_it_nice?] = :yes_it_was

    job.complete!
  end

  def self.create_async_job(classroom_id, handler_class)
    job_id = self.perform_async(student_id)
    JobRepository.new.create(:id => job_id,
                             :handler_class => handler_class,
                             :classroom_id => classroom_id)
  end
end
```

In the above example, `:classroom_id` as passed to the `JobRepository`
is metadata, available for handling job-related tasks.

The sidekiq worker looks up its job (via `repo.find`) as part of its
perform method. It then marks the job as complete with
`job.complete!`, which marks the job as complete.


## Redirecting to the Job Controller

The typical place you would want to call the above `create_async_job`
method is inside a controller. Afterwards, you can redirect the user
to the `JobsController#show` method where they will be shown a
message. For example:

```ruby
class ClassroomController

  # ...

  def a_nice_action
    job = MyNiceClassroomWorker.create_async_job(params[:id], "ClassroomController::NiceJobHandler")
    redirect_to job_path(job.id)
  end

  # ...

end
```

## Job Handler

The `:handler_class` provides a way to customize the way the job
message is displayed and what happens as a job finishes.

```ruby
class ClassroomController

  # ...

  class NiceJobHandler
    def initialize job
      @job = job
    end

    def header
      "The Header on the Spinner Page"
    end

    def message
      "This message appears to the user on the spinner page as body text."
    end

    def next_url controller
      controller.classroom_path(@job.data[:classroom_id])
    end

    def on_ready controller
      if @job.data[:was_it_nice?] == :yes_it_was
        controller.flash[:notice] = "IT WAS SO NICE."
      else
        controller.flash[:alert] = "ERROR being nice"
      end
    end
  end

  # ...

end
```

In the above example, `next_url` and `on_ready` are both passed
references to `JobsController#self`, which lets them do things like
set flash messages, use url helpers, or generally do anything you
normally can do in a controller.

The methods are as follows:

- `initialize` When a job handler is instantiated, the job it is
  handling is provided to `initialize`
- `header` is the header that is displayed on the spinner page.
- `message` is the body text displayed on the spinner page.
- `next_url` is the next url the user should be directed to after the
  spinner page.
- `on_ready` is a hook that is called before the user is redirected
  on to the url provided by `next_url`. In the above example, it is
  used to send a flash message to the user.


## Real Example

The Jobs controller is as-of-this-writing being employed by
`classroom_students_controller.rb` in conjunction with
`student_class_transfer_worker.rb`.
