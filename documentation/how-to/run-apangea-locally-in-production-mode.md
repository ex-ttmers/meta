# Running apangea locally in production mode

Sometimes you need to be able to run apangea locally but with RAILS_ENV = production, not development. The easiest way to do that is:


- Add "production" config to database.yml (I just duplicate my "development" config)
- In config/unicorn.rb, remove the whole block that's in `if rails_env == 'production'`
- In config/environments/production.rb, remove everything configuring logging
- Either set a RAILS_SECRET_TOKEN env var or remove the `if Rails.env.production?` from config/initializers/01_secret_tokens.rb
- Start the app using `RAILS_ENV=production bundle exec foreman start`, optionally with `-f Procfile.Production` depending on if you want to use the production procfile or your normal `Procfile`.
