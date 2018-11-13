Unicorn (our web server) needs Redis connections so that web requests
can enqueue Sidekiq jobs.

This is configured in Unicorns `after_fork` config block in `unicorn.rb` like this:

    Sidekiq.configure_client do |config|
      namespace = ENV['SIDEKIQ_NAMESPACE'] || 'ttm-worker'
      config.redis = { url: ENV['REDIS_URL'], namespace: namespace }
    end

By default, Sidekiq will use up to five Redis connections. Since we
use `ENV['UNICORN_WORKERS']` (4 as of this writing) each Heroku web
dyno will by default use 20 live Redis connections. The production
configuration for now is set:

    Sidekiq.configure_client do |config|
      namespace = ENV['SIDEKIQ_NAMESPACE'] || 'ttm-worker'
      config.redis = { url: ENV['REDIS_URL'], namespace: namespace, size: 2 }
    end

which means each dyno will use 8 live connections (4 workers x 2
connections).
