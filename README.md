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

We'll use Amazon Elastic Beanstalk to deploy this in production and we must translate the `docker-compose.yml` from `zanata-docker-files` to a `Dockerrun.aws.json`. You can use the `Dockerrun_template.aws.json` as reference to create a new one. It's important that the `Dockerrun.aws.json` is the "version 2" as this setup will require a [Multicontainer Docker Configuration](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html).

### Amazon Elastic Beanstalk configuration

On Elastic Beanstalk create an enviroment with the following properties:

- Web server environment
- Preconfigured platform: Multi-container Docker
- Upload you code: Select the `Dockerrun.aws.json`
- Instances: At least a `t2.small`
- EC2 security group: Open the inbout ports for `HTTP` (port `80`), `HTTPS` (port `443`) and `MySQL` (port `3306`)

The `Dockerrun.aws.json` should be uploaded and deployed. You can do it through the "Application Versions page" or through the button "upload and deploy" on the environment's dashboard.

### Database

#### Creating first user

Once the application is deployed for the first time, there will be no users and no way to login. To enable login you must run an SQL query manually on the database (that is why it's important to open the inbound port `3306`). The [example SQL query](https://github.com/zanata/zanata-docker-files/blob/master/zanata-server/conf/admin-user-setup.sql) can be found at the [`zanata-docker-files`](https://github.com/zanata/zanata-docker-files).

Check [MySQL Workbench](https://www.mysql.com/products/workbench/) or any other client compatible with `MariaDB` to run the query. Make sure the first thing you do is to login with the administrator credentials and change the default password.

```sql
-- Username: admin
-- Password: admin1234
INSERT INTO HAccount (id, creationDate, lastChanged, versionNum, enabled, passwordHash, username)
VALUES (1, now(), now(), 1, 1, 'UZMf4PIqtTBGAo9wWKuTpg==', 'admin');

INSERT INTO HPerson (id, creationDate, lastChanged, versionNum, email, `name`, accountId)
VALUES (1, now(), now(), 1, 'admin@noreply.org', 'Administrator', 1);

INSERT INTO HAccountMembership(accountId, memberOf) VALUES (1, 5);
```

#### Data export and import

Running the database on a container without backups can lead to data loss if the Elastic Beanstalk enviroment is rebuilt. To avoid this, before any big operations it's important to backup the data.

Use your favorite MySQL client to connect and dump the tables of your database. I recommend [MySQL Workbench](https://www.mysql.com/products/workbench/) even it complains about compatibility with `MariaDB`.

1. Connect to your database
1. On Workbench 6.3 for linux it shows a "Management" menu on the left sidebar. Click on "Data Export"
1. Select the database you want to export
1. If you are planning to import this data on a clean database (no Zanata has connected to it before), select to "Include Create Schema". If you are planning to restore the data on an empty Zanata project, don't select "Include Create Schema".
1. Select "Export to Self-Contained File"
1. Click on "Start Export"

This will generate a single dump file.

To import the dump, just under "Data Export" item on the sidebar there is a "Data Import/Restore":

1. Select "Import from Self-Contained File"
1. Select the dump file
1. Click on "Start Import"
