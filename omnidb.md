# How-To: Connect to a database

Often when deployments contain SQL databases like Postgresq, Mysql, Mariadb,
we would like to connect to that database and get a sneak peak into data.
There are 2 ways to do it:

- Expose database via URL so that sql clients like command line tools and
  GUIs can connect.
- Have a web UI through which database can be accessed.


This document will explore how to use web UI to access database using OmniDB.
[OmniDB][omnidb] is web tool for managing multiple databases. We can run OmniDB
as a container which connects to the database deployed.

### Configuring a container with web UI

Dockup team has customized OmniDB docker image so that OmniDB can be easily
used with deployments. Edit blueprint and add a container called OmniDB.
We need to configure this container with following env variables:

| Environment Variable   | Description                                            |
|------------------------|--------------------------------------------------------|
| `OMNIDB_TECHNOLOGY`    | Values can be one of `postgresql`, `mariadb`           |
|                        | or `mysql`                                             |
| `OMNIDB_USER`          | Username used to connect to db. Can be admin user also |
| `OMNIDB_PORT`          | Port exposed by database container                     |
| `OMNIDB_HOST`          | Use special env variables, and connect to the database |
|                        | container in the deployment.                           |
| `OMNIDB_DATABASE`      | Name of the database to connect                        |


Once we enter all these values, it looks something like this.

![Container](/images/omnidb-container.png)


### Accessing database

Once blueprint is deployed, we can access web UI as part of deployment.

![Deployment](/images/omnidb-deployment.png)

Click on link next to "OmniDB" and we should see an interface to add `user` and
`pwd`. Please enter `admin` and `admin`.

![Login](/images/omnidb-login.png)

Now, we see OmniDB interface where name of database to access will be on top
right corner.

![Interface](/images/omnidb-interface.png)

Expand Database `+` icon (In this case MySQL). Interface will ask for database
password. Enter the password, and we can now access the database.

![Credentials](/images/omnidb-db-password.png)

Now we can perform CRUD operations using query interface.

![Query interface](/images/omnidb-query-interface.png)

For further information (if required), please drop a mail to
support@getdockup.com


[omnidb]: https://omnidb.org/en/
