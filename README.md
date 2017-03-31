# Dependencies
* Staffjoy PROD is deployed on a docker container running on [Zeit](https://zeit.co/).
* Redis is hosted on [Compose.io](https://app.compose.io/session/new)
* MySQL is hosted on [AWS](https://us-west-2.console.aws.amazon.com/console/home?region=us-west-2#)
* The keepAlive service runs on a cron on a [private server](https://github.com/andrewtlove)
* Logging is done through [Sentry.io](https://sentry.io/auth/login/)

# Deploying to PH Prod

Note: This uses the `--alias` flag, which is soon to be depreciated.

`torus run -- now -t $PH_ZEIT_KEY -e BASE_URL -e ENV -e RECAPTCHA_PRIVATE_KEY -e RECAPTCHA_PUBLIC_KEY -e REDIS_HOST -e REDIS_PORT -e REDIS_PASSWORD -e REDIS_USER -e REDIS_URI -e SECRET_KEY -e SQLALCHEMY_DATABASE_URI -e MANDRILL_API_KEY -e TWILIO_ACCOUNT_SID -e TWILIO_AUTH_TOKEN -e FROM_EMAIL -e PAPERTRAIL -e SENTRY -a staffjoy.partnerhero.com --public`

# Suite, aka Staffjoy V1

[![Build Status](https://travis-ci.org/Staffjoy/suite.svg?branch=master)](https://travis-ci.org/Staffjoy/suite) [![Moonlight](https://img.shields.io/badge/contractors-1-brightgreen.svg)](https://moonlightwork.com/staffjoy) [![Docker Automated build](https://img.shields.io/docker/automated/jrottenberg/ffmpeg.svg)](https://hub.docker.com/r/staffjoy/suite/)

[Staffjoy is shutting down](https://blog.staffjoy.com/staffjoy-is-shutting-down-39f7b5d66ef6#.ldsdqb1kp), so we are open-sourcing our code. This version of our V1, intended for on-demand companies and call centers, has been heavily modified so that Staffjoy customers may continue using the software. 

## Credit

The authors of the original code were [@philipithomas](https://github.com/philipithomas), [@andhess](https://github.com/andhess), and [@bahador](https://github.com/bahador). This is a fork of the internal repository. For security purposes, the Git history has been squashed.

## Licensing

If you are using this repo for commercial purposes, please [review the Highcharts licensing requirements](https://shop.highsoft.com/highcharts?utm_expid=116444199-11.dlP5ft76QI-e766NMF5Rgw.0&utm_referrer=https%3A%2F%2Fshop.highsoft.com%2F) for commercial use, in addition to the `LICENSE` file in this repo.

## Running the repo for production use

### Database Requirements

* Redis
* MySQL

### Environment Variables

This table intends to explain the main requirements specified in `app/config.py`. This configuration file can be manually edited, but be careful to not commit secret information into the git repository. Please explore the config file for full customization info.

Name | Description | Example Format
---- | ----------- | --------------
ENV | "prod", "stage", or "dev" to specify the configuration to use. When running the code, use "prod". | prod
BASE_URL | URL where the code is hosted. | https://suite.staffjoy.com
MANDRILL_API_KEY | API Key for [Mandrill](http://mandrillapp.com) for sending emails. | 
FROM_EMAIL | Email address from which notifications will be sent in Mandrill | team@staffjoy.com
RECAPTCHA_PUBLIC_KEY | Public key for [Recaptcha](https://www.google.com/recaptcha/intro/) | 
RECAPTCHA_PRIVATE_KEY | Private key for Recaptcha |
REDIS_HOST | Host for redis | localhost
REDIS_PORT | Port to connect to Redis on. Defaults to 6379. | 6379
REDIS_DATABASE | If using multiple databases, set the number. Defaults to 0 | 0
SQLALCHEMY_DATABASE_URI | Connection info, including username, password, and database for MySQL | mysql://root:bacon@localhost/dev
SECRET_KEY | A unique, secret key for your application that is used to sign cookie data. Make sure it is the same across instances in an environment. **Setting this key is critical for security.** ([details](https://stackoverflow.com/questions/22463939/demystify-flask-app-secret-key)) | anyLongSecretKeyYouMakeUpAndIsRandom
STATUS_PAGE_ID | Optional page id for a [Status Page](http://statuspage.io) integration |
STATUS_PAGE_API_KEY | Corresponding API key for Statuspage.io | 
INTERCOM_ID | Optional - application ID for an [Intercom](http://intercom.io) app |
INTERCOM_SECRET | Optional - Intercom secret used for signing user hashes | 
INTERCOM_API_KEY | Optional - Intercom API key used for sending events to the Intercom API | 
TWILIO_ACCOUNT_SID | API Key for [Twilio](http://twilio.com) - note that the `TWILIO_NUMBER` variable in `config.py` needs to be updated with sending phone numbers that you own for this to work. | 
TWILIO_AUTH_TOKEN | Twilio secret key | 

## Required and Optional Services

### Cron

The `/api/v2/internal/cron/` endpoint must be triggered every 60 seconds. We are open-sourcing a microserviec to do this soon. 

Alternatively, consider just using a Jenkins job with:

```curl --user API_KEY: http://suite.local/api/v2/internal/cron/```

(*NOTE*: the trailing colon after the API key is required)

### Chomp

Required for calculating shifts from forecasts. [View Chomp's source code on github.com/staffjoy/chomp-decomposition](https://github.com/staffjoy/chomp-decomposition)

### Mobius

Required for assigning shifts. Coming soon.

### How we deployed

* AWS elastic beanstalk to run the Docker containers ([here's our deploy script](https://gist.github.com/philipithomas/190de362654601da43a08f3dc63ce4eb))
* AWS RDS for MySQL
* AWS Elasticache for Redis
* Healthcheck `/health` endpoint to determine instance health
* Terminate SSL at load balancer
* Run Cloudflare upstream
* Run at least three production containers (N + 2) for redundancy
* Stream logs to [Papertrail](http://papertrailapp.com)
* Strict firewalling of Redis for security

### Notes

* It is suggested that you fork this repository if you are using it
* If you do not want anybody from the web signing up for your application and provisioning new organizations, modify `ALLOW_COMPANY_SIGNUPS` in `app/config.py`
* The Staffjoy mobile applications do not work with self-hosted applications.

## Development Setup

Prequisites:

* Install [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
* Install [Vagrant](https://www.vagrantup.com/)

Add your email to the file `user.txt`, e.g. if your email is *lenny@staffjoy.com*:

```
echo "lenny@staffjoy.com" > ./user.txt
```

This auto-registers you on first boot, and makes you an admin.

To boot a dev server for the first time, run:
```
make dev
```

The first time it launches, it will take a long time to download the required base images. After that, required packages and images will be cached. The app will be running at [suite.local](http://suite.local)

To quickly launch a dev server that has already been built, you can use this shortcut command from the project root:

```
vagrant up
vagrant ssh
cd /vagrant
make dev-server
```

Be advised that this does not update any required packages or recompile the less stylesheets, so please run ```make dev``` occasionally.

### Installing new packages

Add new dependencies to the end of `requirements.txt`, then run `make dev`. The dev server is booted, the dependencies are installed, then the boot script immediately runs `pip freeze` to update requirements.txt with the latest version number, and nested dependencies.

### Hard-resetting a dev server

If you run into issues or if you messed up your environment, you can delete its caches with `vagrant destroy -f`. Running `make dev` will take longer as the system re-fetches dependencies.

### Creating a sudo account in dev

After creating a Staffjoy account in your dev environt, use the commands below to access the shell:

```
vagrant ssh
cd /vagrant
make shell
```

Now in the shell window, write and execute the following code:

```
print User.query.get.all() # To find users - don't do this in prod where there are lots of user :-)
u = User.query.get(<id>)
u.sudo = True
db.session.commit()
```

The id will correspond to the id of the user that you just created and will be visible in the URL bar


## Working with the Repo

### Environments

To access the current environment, use:

```
from flask import current_app
print current_app.config.get("ENV") # dev or prod - as defined in config.py.
# Note that stage is treated as prod except for the robots.txt
```

### Working with the database

**Dev** - uses MySQL just like production. To access it, run the following commands from the project root:

```
vagrant ssh
<wait for login>
cd /vagrant
mysql -u root -p
<enter the password "bacon">
use dev;
```

The development MySQL root password is ```bacon```.

#### Database Migrations Summary

1. Make a change to the model (by modifying app/models.py)
2. Generate a migration that spits out the changes that need to be made to the database with `make db-migrate`
3. Apply those generated changes with `make db-deploy`

*Note: Commit the migration files*

#### Dev

While your vagrant machine is running, SSH into the instance with `vagrant ssh`. Then in `/vagrant/`, run `make db-migrate` to generate a migration. To apply the database change, you can either run `make db-deploy`, or opt for a standard `make-dev`.

##### If this doesn't work

It's easier to wipe a database than try and do a rollback. Log into the Dev MySQL Instance and run this command to wipe it:

```
drop database dev;
create database dev;
```

Exit MySQL and now run `make db-deploy` to rebuild it. You will have to create a new sudo user with the Shell Context.

### Shell Context

You can run `python main.py shell` and interact directly with the app, users, etc.

e.g.
```
python manage.py shell
u = User()
u.password = 'cat'
u.password_hash # See hash
u.verify_password('cat')
u.verify_password('dog')
db.session.commit()
```

This is useful in dev environments for clearing out the database or going into production web servers to flag a user as "sudo".

## API

Public docs forthcoming

## Tokens

Tokens are time-based tokens with a default life of 6 hours. API keys are permanent until revoked. 

We authenticate using HTTP basic auth, where the username is the token and the password is blank. To do this with curl, use this command: (noting that the trailing colon after the token is important because it means "no password")

``` curl -u TOKEN: https://www.staffjoy.com/api/v2/ ```

You can always get a new token for your logged-in user at [/auth/api-token](https://www.staffjoy.com/auth/api-token).

## Specs

Our API uses JSON. Response data is in the `data` field, resources on the object are given in the `resources` field, and other metadata - like `limit` or `offset` may be provided.

## Formatting 

This library uses the [Google YAPF](https://github.com/google/yapf) library to enforce PEP-8. Using it is easy - run `make fmt` to format your code inline correctly. Failure to do this will result in your build failing. You have been warned.

To disable YAPf around code that you do not want changed, wrap it like this:

```
# yapf: disable
FOO = {
    # ... some very large, complex data literal.
}

BAR = [
    # ... another large data literal.
]
# yapf: enable
```

## Miscellaneous


##  Resolving database issues

If you end up with multiple heads, do:

```
python main.py db heads
# Using the two hashes
python main.py db merge <hash1> <hash2>
```

## Other Features

* Twilio IVR
* StatusPage.io integration
* Mobile application detection via headers

## Known issues

* The development environment works, but really needs a ground-up rewrite to something like Docker Compose. (It hasn't been extensively modified since before Staffjoy was a full-time job!)
* The tests use `current_app` rather than the generator `create_app`. This should be corrected.
* The session management system, when reading a session from a cookie, recreates the session. This means that the list of active sessions is longer than expected (though still secure).
