# Deploying to Heroku

Heroku is a Platform as a Service (PaaS) that allows you to deploy and run applications without managing infrastructure. This guide explains how to deploy a self-hosted Gumroad instance on Heroku.

## Prerequisites

Before you begin, ensure you have:

- A Heroku account
- The [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) installed
- `git` installed locally
- A fork of the Gumroad repository

## Step 1: Create a Heroku App

From the root of the Gumroad project, create a new Heroku application:

```bash
heroku create my-gumroad-app
```

Replace `my-gumroad-app` with a unique name.

## Step 2: Configure Buildpacks

Gumroad requires both Node.js (for asset compilation) and Ruby. Ensure the buildpacks are configured in the correct order:

```bash
heroku buildpacks:set heroku/nodejs
heroku buildpacks:add heroku/ruby
```

You can verify the order with:

```bash
heroku buildpacks
```

## Step 3: Provision Add-ons

Gumroad requires a relational database and Redis.

### PostgreSQL

```bash
heroku addons:create heroku-postgresql
```

### Redis

```bash
heroku addons:create heroku-redis
```

Heroku will automatically inject the appropriate `DATABASE_URL` and `REDIS_URL` environment variables.

## Step 4: Configure Environment Variables

Set the required environment variables. At minimum, you will need:

```bash
heroku config:set RAILS_ENV=production
heroku config:set RAILS_MASTER_KEY=<your-rails-master-key>
```

The `RAILS_MASTER_KEY` value should match the contents of your local `config/master.key`.

Additional environment variables may be required depending on your setup, such as AWS credentials, Stripe API keys, and email provider configuration. Refer to `.env.example` for a complete list.

## Step 5: Deploy

Push your code to Heroku:

```bash
git push heroku main
```

If your default branch is not `main`, push the appropriate branch instead.

## Step 6: Database Setup

Once the deployment finishes, run the database migrations:

```bash
heroku run bundle exec rails db:migrate
```

## Step 7: Enable Background Workers

Gumroad uses Sidekiq for background job processing. Enable a worker dyno:

```bash
heroku ps:scale worker=1
```

You may scale this up later based on workload.

## Step 8: Verify Deployment

Open your application in the browser:

```bash
heroku open
```

If the application boots successfully, the deployment is complete.

## Troubleshooting

If something goes wrong, check the logs:

```bash
heroku logs --tail
```

Common issues include:

- Missing or incorrect `RAILS_MASTER_KEY`
- Asset compilation failures due to missing Node/Yarn
- Redis not provisioned or misconfigured
- Sidekiq not running (worker dyno not scaled)

## Notes

- Heroku dynos use an ephemeral filesystem. Do not rely on local disk storage.
- Ensure all file uploads are configured to use external storage (for example, Amazon S3).
- Production performance may require tuning dyno sizes and worker counts.
