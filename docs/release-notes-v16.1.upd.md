# Percona Distribution for PostgreSQL 16.1 Update (2024-01-18)

[Installation](installing.md){.md-button}

Percona Distribution for PostgreSQL is a solution with the collection of tools from PostgreSQL community that are tested to work together and serve to assist you in deploying and managing PostgreSQL. The aim of Percona Distribution for PostgreSQL is to address the operational issues like High-Availability, Disaster Recovery, Security, Observability, Spatial data handling, Performance and Scalability and others that enterprises are facing.

This update of Percona Distribution for PostgreSQL includes the new version of [`pg_stat_monitor` 2.0.4 :octicons-link-external-16:](https://docs.percona.com/pg-stat-monitor/release-notes/2.0.4.html) that fixes the issue with the extension causing the deadlock in the Percona Operator for PostgreSQL when executing the `pgsm_store` function.
