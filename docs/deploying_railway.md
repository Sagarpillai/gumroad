# Deploying to Railway

Railway is a modern infrastructure platform that makes it easy to deploy and run applications in the cloud. This guide explains how to deploy a self-hosted Gumroad instance on Railway.

## Prerequisites

Before you begin, ensure you have:

- A Railway account
- The [Railway CLI](https://docs.railway.app/guides/cli) installed (optional — the web dashboard can also be used)
- `git` installed locally
- A fork of the Gumroad repository

---

## Step 1: Create a New Railway Project

1. Sign in to the Railway dashboard.
2. Click **New Project**.
3. Choose **Deploy from GitHub Repo**.
4. Select your Gumroad fork and the branch you want to deploy.

Railway will automatically detect the Rails application and begin the initial build.

---

## Step 2: Add Required Services

Gumroad requires both a relational database and Redis.

### PostgreSQL

From the Railway project dashboard:

1. Click **Add Service**
2. Select **PostgreSQL**

Railway will automatically provide a `DATABASE_URL` environment variable.

### Redis

1. Click **Add Service**
2. Select **Redis**

Railway will automatically provide a `REDIS_URL` environment variable.

---

## Step 3: Configure Environment Variables

In the Railway dashboard, open your application service and configure the required environment variables.

At minimum, you will need:

- `RAILS_ENV=production`
- `RAILS_MASTER_KEY=<your-rails-master-key>`

The `RAILS_MASTER_KEY` value should match the contents of your local `config/master.key`.

Additional environment variables may be required depending on your setup, such as:

- AWS credentials (for file uploads)
- Stripe API keys
- Email provider configuration

Refer to `.env.example` for a complete list of configuration options.

---

## Step 4: Configure the Start Command

Railway automatically detects Rails applications, but you should ensure the correct start command is configured:

```bash
bundle exec rails server -b 0.0.0.0 -p $PORT
```

This can be set in the **Start Command** field of the service settings.

---

## Step 5: Run Database Migrations

After the first successful deployment, run database migrations.

Using the Railway CLI:

```bash
railway run bundle exec rails db:migrate
```

Or via the Railway dashboard terminal.

---

## Step 6: Enable Background Workers

Gumroad uses Sidekiq for background job processing. Create a separate service for workers:

1. Duplicate the main application service.
2. Set the start command to:

```bash
bundle exec sidekiq
```

3. Ensure the worker service has access to the same environment variables and Redis instance.

You can scale workers based on workload requirements.

---

## Step 7: Verify Deployment

Once the deployment completes, open the generated Railway domain from the dashboard and verify that the application loads successfully.

---

## Troubleshooting

To inspect logs, use the Railway dashboard or CLI:

```bash
railway logs
```

Common issues include:

- Missing or incorrect `RAILS_MASTER_KEY`
- Database migrations not run
- Redis service not connected
- Sidekiq not running or misconfigured

---

## Notes

- Railway provides ephemeral filesystems. Do not rely on local disk storage.
- Ensure all file uploads are configured to use external storage (for example, Amazon S3).
- Production performance may require tuning service resources and worker counts.
