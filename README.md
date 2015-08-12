# Capistrano::Maintenance

## Installation

Add this line to your application's Gemfile:

    gem 'capistrano', '~> 3.0'
    gem 'capistrano-maintenance', github: "capistrano/maintenance", require: false 


And then execute:

    $ bundle

Or install it yourself:

    $ gem install capistrano-maintenance

## Usage

Before using the maintenance tasks, you need to configure your webserver. How you do this depends on how your server is configured, but the following examples should help you on your way.

Application servers such as [Passenger](https://www.phusionpassenger.com) and [unicorn](http://unicorn.bogomips.org) requires you to set your public web directory to `current/public`. Both examples below are compatible with this setup.

Here is an example config for **nginx**. Note that this is just a part of the complete config, and will probably require modifications.

```
error_page 503 @503;

# Return a 503 error if the maintenance page exists.
if (-f /var/www/domain.com/shared/public/system/maintenance.html) {
  return 503;
}

location @503 {
  # Serve static assets if found.
  if (-f $request_filename) {
    break;
  }

  # Set root to the shared directory.
  root /var/www/domain.com/shared/public;
  rewrite ^(.*)$ /system/maintenance.html break;
}
```

And here is an example config for **Apache**:

```
ErrorDocument 503 /system/maintenance.html
RewriteEngine On
RewriteCond %{REQUEST_URI} !.(css|gif|jpg|png)$
RewriteCond %{DOCUMENT_ROOT}/system/maintenance.html -f
RewriteCond %{SCRIPT_FILENAME} !maintenance.html
RewriteRule ^.*$ - [redirect=503,last]
```

You can now require the gem in your `Capfile`:

    require 'capistrano/maintenance'

### Enable task

Present a maintenance page to visitors. Disables your application's web
interface by writing a "#{maintenance_basename}.html" file to each web server. The
servers must be configured to detect the presence of this file, and if
it is present, always display it instead of performing the request.

By default, the maintenance page will just say the site is down for
"maintenance", and will be back "shortly", but you can customize the
page by specifying the REASON and UNTIL environment variables:

    cap maintenance:enable REASON="hardware upgrade" UNTIL="12pm Central Time"

You can use a different template for the maintenance page by setting the
`:maintenance_template_path` variable in your deploy.rb file. The template file
should either be a plaintext or an erb file.

Further customization will require that you write your own task.

### Disable task

    cap maintenance:disable

Makes the application web-accessible again. Removes the
"#{maintenance_basename}.html" page generated by maintenance:disable, which (if your
web servers are configured correctly) will make your application web-accessible again.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
