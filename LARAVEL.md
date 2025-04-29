# Laravel Support in Stedding

Stedding has been extended to support Laravel applications alongside WordPress sites. This enables you to use the same robust server provisioning and zero-downtime deployment workflow for both Laravel and WordPress projects.

## How It Works

Laravel support is implemented by:

1. Adding Laravel-specific roles (`laravel-setup` and `laravel-install`) that handle Laravel's different requirements
2. Configuring Nginx to work with Laravel's `/public` web root
3. Using Artisan commands instead of WP-CLI
4. Supporting Laravel's `.env` file format and environment variables
5. **Conditional provisioning:** Trellis roles and tasks now use the `site_type` property to ensure only the correct tasks run for each site (WordPress or Laravel). This prevents WordPress tasks from running on Laravel sites and vice versa.

## Usage

### 1. Define Laravel Sites

In your `wordpress_sites.yml` file, add a `site_type: laravel` property to sites that are Laravel projects:

```yaml
wordpress_sites:
  laravel-app.com:
    site_type: laravel  # This identifies the site as a Laravel project
    site_hosts:
      - canonical: laravel-app.test
        redirects:
          - www.laravel-app.test
    local_path: ../laravel # path targeting local Laravel project directory
    admin_email: admin@laravel-app.test  # Required but unused for Laravel
    ssl:
      enabled: false
      provider: self-signed
    cache:
      enabled: false
    nginx_includes:
      - roles/laravel-setup/templates/laravel-site.conf.j2  # REQUIRED for Laravel sites
    env:
      APP_NAME: "Laravel App"
      APP_ENV: local
      APP_DEBUG: true
      APP_URL: http://laravel-app.test
      MAIL_HOST: mailhog
      MAIL_PORT: 1025
      CACHE_DRIVER: file
      SESSION_DRIVER: file
      QUEUE_DRIVER: sync
```

### 2. Customize Nginx (Optional)

If you need additional Nginx configuration beyond the default Laravel setup, you can create custom Nginx configurations for your Laravel site at:

```
nginx-includes/laravel-app.com/custom-rules.conf.j2
```

These will be included automatically alongside the main Laravel configuration.

### 3. Provision & Deploy

Use the standard Stedding commands:

```bash
# For local development
trellis vm start

# For remote servers, provision first
ansible-playbook server.yml -e env=production

# Deploy site
ansible-playbook deploy.yml -e "site=laravel-app.com env=production"
```

## What's Different for Laravel?

- Web root is `current/public` instead of `current/web`
- Using Laravel's scheduler instead of WP Cron
- Running Artisan commands during deployment (migrations, key generation, etc.)
- Laravel-specific Nginx configuration
- Laravel-friendly `.env` file formatting
- **Conditional provisioning:** Only the correct tasks run for each site type, based on the `site_type` property.

## Advanced Options

In your site configuration, you can use these Laravel-specific options:

- `run_migrations: false` - Skip database migrations during deployment
- `cache_config: false` - Skip config caching during deployment

## Example: WordPress and Laravel in One Project

You can define both WordPress and Laravel sites in the same `wordpress_sites.yml`:

```yaml
wordpress_sites:
  my-wp-site.com:
    # No site_type or site_type: wordpress (default)
    ...
  laravel-app.com:
    site_type: laravel
    ...
```

Trellis will now only run WordPress roles for WordPress sites and Laravel roles for Laravel sites.