## How to create a parent locally

1. Make sure you are running [mailcatcher](https://github.com/thinkthroughmath/apangea#emails-in-development).
2. Make sure your local procfile for sidekiq includes the `email` queue.
3. Visit [localhost:5000/customers/1/registrations/new](http://localhost:5000/customers/1/registrations/new) (where `/1/` is the id of a `Customer` in your local apangea).
4. Sign up with whatever name and email you like...
5. Check [your mailcatcher inbox](http://127.0.0.1:1080/).
6. Follow the link in the email to complete registration.
7. Add students as needed.

