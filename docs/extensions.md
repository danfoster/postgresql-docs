# Extensions

Percona Distribution for PostgreSQL includes a set of extensions that have been tested to work together. These extensions enable you to efficiently solve essential practical tasks to operate and manage PostgreSQL.

The set of extensions includes the following:

* [PostgreSQL contrib modules and utilities](contrib.md)
* Extensions authored by Percona:
  
    * [`pg_stat_monitor`](pg-stat-monitor.md)

* [Third-party components](third-party.md)

Additionally, see the list of [PostgreSQL software](https://www.percona.com/services/support/support-tiers-postgresql) covered by Percona Support.

## Install an extension

To use an extension, install it. Run the [`CREATE EXTENSION`](https://www.postgresql.org/docs/current/static/sql-createextension.html) command on the PostgreSQL node where you want the extension to be available. 

The user should be a superuser or have the `CREATE` privilege on the current database to be able to run the [`CREATE EXTENSION`](https://www.postgresql.org/docs/current/static/sql-createextension.html) command. Some extensions may require additional privileges depending on their functionality. To learn more, check the documentation for the desired extension.
