# Strawbees Zanata Server

[Zanata](http://zanata.org/) is a web-based translation platform for translators, content creators and developers to manage localisation projects.

We choose to host our own server to be independent of Zanata's official server downtimes. Thankfully this is very easy to do as Zanata provides [Docker files](https://github.com/zanata/zanata-docker-files) and a [Docker image](https://hub.docker.com/r/zanata/server/).

## Running locally

To run Zanata locally it should be easy:

1. Make sure you have `docker` and `docker-compose` installed and ready to use.
1. Clone Zanata's Docker files repo: `git clone https://github.com/zanata/zanata-docker-files`
1. Navigate to the folder `zanata-docker-files/zanata-server`
1. Create a `.env` file with the following environment variables:
	- `ZANATA_PORT`: What port the server should run
	- `ZANATA_VERSION`: Usually the value for this variable should be `latest`
	- `ZANATA_DAEMON_MODE`: Whether zanata should run in the background (if set to false you will see all the application logs on the command line)
	- `ZANATA_MYSQL_USER`, `ZANATA_MYSQL_PASSWORD`, and `ZANATA_MYSQL_DATABASE`: MySQL database credentials and database name (We'll actually use MariaDB). This is required.
	- `ZANATA_MAIL_HOST`, `ZANATA_MAIL_PORT`, `ZANATA_MAIL_USERNAME` and `ZANATA_MAIL_PASSWORD`: SMTP credentials. This is required for new users to sign up.
	- `ZANATA_MAIL_TLS`, `ZANATA_MAIL_SSL`: Whether use encrypted connection for emails (all no by default).
1. Run `docker-compose -f docker-compose.yml up`

This will spawn 2 docker instances: One for the database and other for the Zanata server.

## Deploying

We'll use Amazon Elastic Beanstalk to deploy this in production and we must translate the `docker-compose.yml` from `zanata-docker-files` to a `Dockerrun.aws.json`. It's important that the `Dockerrun.aws.json` is the "version 2" as this setup will require a [Multicontainer Docker Configuration](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html).
