# heroku-buildpack-rails-assets-sync

Sync rails precompiled assets to S3 using [s3sync-rust](https://github.com/nidor1998/s3sync). Syncing is skipped if CDN_HOST is not set or empty.

## Requirements

* AWS credentials (AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY) are set as config variables and have appropriate credentials for writing the the S3 bucket.
* the CDN_HOST config variable is used as  Rails.application.config.asset_host, e.g.

  ```ruby
  # Enable serving of images, stylesheets, and JavaScripts from CDN_HOST if set.
  config.asset_host = ENV.fetch("CDN_HOST", false)
  ```

### Assumptions

* S3 bucket is eponymously named after the `CDN_HOST` config variable. e.g. if `CDN_HOST=cdn.acme.net` then sync target is `s3://cdn.acme.net/` by default. If a custom bucket name is required, provide the `CDN_HOST_BUCKET` config variable.
* assets have been compiled under `public/assets` -- ensure this buildpack runs after the ruby buildpack.

## Setup Instructions

1. Add this buildpack _after_ the ruby buildpack:

   ```sh
   heroku buildpacks:add https://github.com/starburstlabs/heroku-buildpack-rails-assets-sync
   ```

2. Set required environment variables:

   ```sh
   heroku config:set CDN_HOST=cdn.yourdomain.com
   heroku config:set AWS_ACCESS_KEY_ID=your_access_key
   heroku config:set AWS_SECRET_ACCESS_KEY=your_secret_key
   heroku config:set AWS_DEFAULT_REGION=us-east-1

   # set CDN_HOST_BUCKET if bucket name is different than CDN_HOST
   # heroku config:set CDN_HOST_BUCKET=your-s3-bucket-name
   ```

## Usage Example

After setup, the buildpack will automatically sync your Rails assets during deployment:

```bash
# Deploy your application
git push heroku main

# The buildpack will:
# 1. Check if CDN_HOST is set
# 2. Verify AWS credentials
# 3. Sync public/assets to s3://cdn.yourdomain.com/
```
