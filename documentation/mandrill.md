# Mandrill: the Good Parts

Mandrill is an email sending service we use. At time of writing, we've
used it for implementing weekly district usage report
distribution. There is a lot to sending email, so let's get started.

## FAQ:

### Q: 'Transactional'? What??

I dunno. As far as I can tell, in the context of email marketing
(from which Mandrill was sired), transactional means "anything that's
not part of an email campaign", which *they* state as "anything
triggered by a user's action". This is in turn slightly confusing
because our emails aren't based upon 'actions', they are regular,
weekly emails with custom content.

TL;DR: Don't worry about 'transactional', Mandrill does what we want.

### Q: Why I'm leaving ActionMailer, or, how does ActionMailer factor into using Mandrill?

As we're using it now, it doesn't. We are completely bypassing it.
For more, see later section "I am Mandrill and So Can You".


### Q: What about 'unsubscribe'?

We're using Mandrill to handle this for us. It inserts an unsubscribe
link into the footer of emails we send. If a user clicks that link,
they will be added to a 'blacklist' maintained internally by
Mandrill. Any future emails we try to send through Mandrill to that
address will be rejected by Mandrill.

We can manually remove users from this 'blacklist', though this costs
us some Mandrill 'reputation', <strike>which is a little unclear to me of its
importance</strike> which
[impacts the rate of which we can send emails](http://help.mandrill.com/forums/22011078-Quota-and-Reputation-System),
but at any rate if users have unsubscribed and want to
resubscribe this will happen by removing them from the blacklist.

## I am Mandrill and So Can You

So, you want to know how Mandrill works from an overall perspective?

- Mandrill holds the email's template; Apangea specifies the values
  that should be inserted into the templates.

- Apangea generates JSON which is sent through the Mandrill gem
  to Mandrill's servers through their "REST-inspired" API.

  - This connection happens through the class `MandrillGateway`, found
    in `lib`.

  - The environment variable `MANDRILL_API_KEY` must be set for Mandrill
    to accept the emails.

- Mandrill does the actual emailing, manages things like 'unsubscribe'
  links, and anything else that is important for defeating the
  many-headed-hydra that is email infrastructure administration. Yay,
  computers.

- The Mandrill API key is stored in LastPass. See Jeff, Tim, Andre, or
  someone else who probably would be able to give you access.

## HTML Email Considered Harmful

Building the HTML in email is complicated, serious business. In light
of that, there is a nice middleman app/repository that should help
with inline css rules, embedding mobile css rules, and other malarkey.

Please find it here: https://github.com/thinkthroughmath/email-templates

Oh, there is a super-awesome-amazing tool called
Litmus (https://litmus.com/) that helps you test your emails. Its
kinda like browserstack, but for email layouts.

### Keep Calm and Handlebars On

Templating inside html email is handled by Handlebars. We send the
values over to Mandrill, and it fills them in. Pretty simple, but it
might be a little un-intuitive.

## Mandrill: Test all the Things

Testing email can be a tricky because there is always the fear that an
email could slip out and get sent to a real user.
Steve and I were talking about the fear of testing this and wanting to
be absolutely sure that in the midst of testing we would not be sending
emails to actual customers. We identified two cases which impact the
safety of sending emails.

1. Running code that 'sends' emails potentially accidentally while doing
   QA/testing on farms.

   For this, our solution is to have all envs except production use the
   'test' mandrill API key. In this case, the mandrill api logs will
   help validate that things were called correctly.

2. A dev may want to receive the actual emails that would be sent
   otherwise to users. In this case, the 'test' key won't work because
   it doesn't actually send emails. In this commit, we provide a way to
   override the users email address via an environment variable.

   Thus, if a user would want to test email sending functionality by
   receiving the actual email in their inbox, they would need to:

   1. Set the `MANDRILL_OVERRIDING_EMAIL` environment variable to be
      the test email address in whatever environment they will be using.

   2. Set the `MANDRILL_API_KEY` environment variable to be the actual,
      live api key.

## Other Cliches I Couldn't Work Into This Document
- 1. x. 2. ???. 3. Profit!!
- like you do or like one does
- x for fun and profit
- how stop worrying and learn to love x
- i for one welcome our x overlord
- reports of the death of x have been greatly exaggerated
- x is Dead! Long Live x!
- unreasonable effectiveness of x
- why i'm leaving x
- regular reminder of x
