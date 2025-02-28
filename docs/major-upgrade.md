# Upgrading Percona Distribution for PostgreSQL from 12 to 13

This document describes the in-place upgrade of Percona Distribution for PostgreSQL using the `pg_upgrade`
tool.

!!! important

    When running a major upgrade on **RHEL 8 and compatible derivatives**, consider the following:

    Percona Distribution for PostgreSQL 16.3, 15.7, 14.12, 13.15 and 12.18 include `llvm` packages 16.0.6, while its previous versions 16.2, 15.6, 14.11, 13.14, and 12.17 include `llvm` 12.0.1. Since `llvm` libraries differ and are not compatible, the direct major version upgrade from 15.6 to 16.3 may cause issues. 

    To ensure a smooth upgrade path, follow these steps:

    * Upgrade to the latest minor version within your current major version (e.g., from 12.19 to 12.19).
    * Then, perform the major upgrade to your desired version (e.g., from 12.19 to 13.15).


The in-place upgrade means installing a new version without removing the old version and keeping the data files on the server.

!!! admonition "See also"

    `pg_upgrade` Documentation:

    [https://www.postgresql.org/docs/13/pgupgrade.html](https://www.postgresql.org/docs/13/pgupgrade.html)

Similar to installing, we recommend you to upgrade Percona Distribution for PostgreSQL from Percona repositories.

!!! important

    A major upgrade is a risky process because of many changes between versions and issues that might occur during or after the upgrade. Therefore, make sure to back up your data first. The backup tools are out of scope of this document. Use the backup tool of your choice.

The general in-place upgrade flow for Percona Distribution for PostgreSQL is the following:


1. Install Percona Distribution for PostgreSQL 13 packages.


2. Stop the PostgreSQL service.


3. Check the upgrade without modifying the data.


4. Upgrade Percona Distribution for PostgreSQL.


5. Start PostgreSQL service.


6. Execute the  **analyze_new_cluster.sh** script to generate statistics
so the system is usable.


7. Delete old packages and configuration files.

The exact steps may differ depending on the package manager of your operating system.

## On Debian and Ubuntu using `apt`

Run **all** commands as root or via **sudo**:
{.power-number}

1. Install Percona Distribution for PostgreSQL 13 packages.


    * [Install percona-release :octicons-link-external-16:](https://docs.percona.com/percona-software-repositories/installing.html)
    
    * Enable Percona repository:

      ```{.bash data-prompt="$"}
      $ sudo percona-release setup ppg-13
      ```

    * Install Percona Distribution for PostgreSQL 13 package:

      ```{.bash data-prompt="$"}
      $ sudo apt install percona-postgresql-13
      ```

2. Stop the `postgresql` service.

    ```{.bash data-prompt="$"}
    $ sudo systemctl stop postgresql.service
    ```

    This stops both Percona Distribution for PostgreSQL 12 and 13.


3. Run the database upgrade.


    * Log in as the `postgres` user.

      ```{.bash data-prompt="$"}
      $ sudo su postgres
      ```


    * Change the current directory to the `tmp` directory where logs and some scripts will be recorded:

      ```{.bash data-prompt="$"}
      $ cd tmp/
      ```


    * Check the ability to upgrade Percona Distribution for PostgreSQL from 12 to 13:

      ```{.bash data-prompt="$"}
      $ /usr/lib/postgresql/13/bin/pg_upgrade \
      --old-datadir=/var/lib/postgresql/12/main \
      --new-datadir=/var/lib/postgresql/13/main  \
      --old-bindir=/usr/lib/postgresql/12/bin  \
      --new-bindir=/usr/lib/postgresql/13/bin  \
      --old-options '-c config_file=/etc/postgresql/12/main/postgresql.conf' \
      --new-options '-c config_file=/etc/postgresql/13/main/postgresql.conf' \
      --check
      ```

      The `--check` flag here instructs `pg_upgrade` to only check the upgrade without changing any data.

      **Sample output**

      ```{.text .no-copy}
      Performing Consistency Checks
      -----------------------------
      Checking cluster versions                                   ok
      Checking database user is the install user                  ok
      Checking database connection settings                       ok
      Checking for prepared transactions                          ok
      Checking for reg* data types in user tables                 ok
      Checking for contrib/isn with bigint-passing mismatch       ok
      Checking for tables WITH OIDS                               ok
      Checking for invalid "sql_identifier" user columns          ok
      Checking for presence of required libraries                 ok
      Checking database user is the install user                  ok
      Checking for prepared transactions                          ok

      *Clusters are compatible*
      ```


    * Upgrade the Percona Distribution for PostgreSQL

      ```{.bash data-prompt="$"}
      $ /usr/lib/postgresql/13/bin/pg_upgrade \
      --old-datadir=/var/lib/postgresql/12/main \
      --new-datadir=/var/lib/postgresql/13/main  \
      --old-bindir=/usr/lib/postgresql/12/bin  \
      --new-bindir=/usr/lib/postgresql/13/bin  \
      --old-options '-c config_file=/etc/postgresql/12/main/postgresql.conf' \
      --new-options '-c config_file=/etc/postgresql/13/main/postgresql.conf' \
      --link
      ```

      The  `--link` flag creates hard links to the files on the old version cluster so you don’t need to copy data.

      If you don’t wish to use the `--link` option, make sure that you have enough disk space to store 2 copies of files for both old version and new version clusters.


    * Go back to the regular user:


      ```{.bash data-prompt="$"}
      $ exit
      ```


    * The Percona Distribution for PostgreSQL 12 uses the `5432` port while the Percona Distribution for PostgreSQL 13 is set up to use the `5433` port by default. To start the Percona Distribution for PostgreSQL 13, swap ports in the configuration files of both versions.

      ```{.bash data-prompt="$"}
      $ sudo vim /etc/postgresql/13/main/postgresql.conf
      $ port = 5433 # Change to 5432 here
      $ sudo vim /etc/postgresql/12/main/postgresql.conf
      $ port = 5432 # Change to 5433 here
      ```


4. Start the `postgreqsl` service.

    ```{.bash data-prompt="$"}
    $ sudo systemctl start postgresql.service
    ```


5. Check the `postgresql` version.

    * Log in as a postgres user
 
       ```{.bash data-prompt="$"}
       $ sudo su postgres
       ```

    * Check the database version
    
       ```{.bash data-prompt="$"}
       $ psql -c "SELECT version();"
       ```


6. After the upgrade, the Optimizer statistics are not transferred to the new cluster. Run the `vaccumdb` command to analyze the new cluster:

    ```{.bash data-prompt="$"}
    $ /usr/lib/postgresql/13/bin/vacuumdb --all --analyze-in-stages
    ```

7. Delete the old cluster's data files:
    
    ```{.bash data-prompt="$"}
    $ ./delete_old_cluster.sh
    $ sudo rm -rf /etc/postgresql/13/main
    $ #Logout
    $ exit
    ```

## On Red Hat Enterprise Linux and derivatives using `yum`

Run **all** commands as root or via **sudo**:
{.power-number}

1. Install Percona Distribution for PostgreSQL 13 packages


    * [Install percona-release :octicons-link-external-16:](https://docs.percona.com/percona-software-repositories/installing.html)
    
    * Enable Percona repository:

       ```{.bash data-prompt="$"}
       $ sudo percona-release setup ppg-13
       ```


    * Install Percona Distribution for PostgreSQL 13:

       ```{.bash data-prompt="$"}
       $ sudo yum install percona-postgresql13-server
       ```

2. Set up Percona Distribution for PostgreSQL 13 cluster

   * Log is as the postgres user

      ```{.bash data-prompt="$"}
      $ sudo su postgres
      ```

   * Set up locale settings

      ```{.bash data-prompt="$"}
      $ export LC_ALL="en_US.UTF-8"
      $ export LC_CTYPE="en_US.UTF-8"
      ```

   * Initialize cluster with the new data directory

      ```{.bash data-prompt="$"}
      $ /usr/pgsql-13/bin/initdb -D /var/lib/pgsql/13/data
      ```

3. Stop the `postgresql` 12 service

    ```{.bash data-prompt="$"}
    $ sudo systemctl stop postgresql-12
    ```

4. Run the database upgrade.


    * Log in as the `postgres` user

       ```{.bash data-prompt="$"}
       $ sudo su postgres
       ```

    * Check the ability to upgrade Percona Distribution for PostgreSQL from 12 to 13:

       ```{.bash data-prompt="$"}
       $ /usr/pgsql-13/bin/pg_upgrade \
       --old-bindir /usr/pgsql-12/bin \
       --new-bindir /usr/pgsql-13/bin  \
       --old-datadir /var/lib/pgsql/12/data \
       --new-datadir /var/lib/pgsql/13/data \
       --check
       ```

       The `--check` flag here instructs `pg_upgrade` to only check the upgrade without changing any data.

       **Sample output**

       ```{.text .no-copy}
       Performing Consistency Checks
       -----------------------------
       Checking cluster versions                                   ok
       Checking database user is the install user                  ok
       Checking database connection settings                       ok
       Checking for prepared transactions                          ok
       Checking for reg* data types in user tables                 ok
       Checking for contrib/isn with bigint-passing mismatch       ok
       Checking for tables WITH OIDS                               ok
       Checking for invalid "sql_identifier" user columns          ok
       Checking for presence of required libraries                 ok
       Checking database user is the install user                  ok
       Checking for prepared transactions                          ok

       *Clusters are compatible*
       ```


    * Upgrade the Percona Distribution for PostgreSQL

       ```{.bash data-prompt="$"}
       $ /usr/pgsql-13/bin/pg_upgrade \
       --old-bindir /usr/pgsql-12/bin \
       --new-bindir /usr/pgsql-13/bin  \
       --old-datadir /var/lib/pgsql/12/data \
       --new-datadir /var/lib/pgsql/13/data \
       --link 
       ```

       The  `--link` flag creates hard links to the files on the old version cluster so you don’t need to copy data.
       If you don’t wish to use the `--link` option, make sure that you have enough disk space to store 2 copies of files for both old version and new version clusters.


5. Start the `postgresql` 13 service.

    ```{.bash data-prompt="$"}
    $ systemctl start postgresql-13
    ```

6. Check `postgresql` status

    ```{.bash data-prompt="$"}
    $ systemctl status postgresql-13
    ```


7. After the upgrade, the Optimizer statistics are not transferred to the new cluster. Run the `vaccumdb` command to analyze the new cluster:


    * Log in as the postgres user

       ```{.bash data-prompt="$"}
       $ sudo su postgres
       ```

    * Run the `vaccumdb` command

       ```{.bash data-prompt="$"}
       $ /usr/pgsql-13/bin/vacuumdb --all --analyze-in-stages
       ```


8. Delete Percona Distribution for PostgreSQL 12 configuration files

    ```{.bash data-prompt="$"}
    $ ./delete_old_cluster.sh
    ```

9. Delete Percona Distribution old data files
    
    ```{.bash data-prompt="$"}
    $ rm -rf /var/lib/pgsql/12/data
    ```
