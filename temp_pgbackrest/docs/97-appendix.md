## Appendix

### Configuration Reference

You can use _pgBackRest_ entirely with command-line parameters, but a configuration file is more practical for complex installations when configuring multiple options instantly. The default location for the configuration file is `/etc/pgbackrest/pgbackrest.conf`. If the configuration file does not exist in that location, then the old `/etc/pgbackrest.conf` file will be loaded instead (if it exists).

#### Global

- Use the global section of the configuration file to set the default values. The stanza-level configuration settings overrides any global configuration setting. 

- The global section also allows you to define values that will be used when invoking specific commands. For example, the following configuration setting would specify the compress level for the `archive-push` command:

```
[global:archive-push]
compress-level=3
```

#### Stanza

- A **stanza** is a database cluster configuration that defines where the database cluster is located, how to take a backup of the database cluster, available archiving options, etc. Most database servers will only have one database cluster and, therefore, one stanza; whereas backup servers will have a stanza for each database cluster that needs to be backed up.

- Because the stanza name  will be used for the primary and all replicas, it is recommended to choose a name that describes the cluster's databases and actual function, such as app or dw, rather than the local instance name, such as main or prod.

- The stanza level settings overrides any global configuration settings.

#### Command Line Arguments and Environment Variables

You can also configure options as command line argument or environment variable.

- To use a non-boolean argument on the command line, simply prefix the option with a double dash.

- To set a boolean option to false on the command line, use the `--no-` prefix.

- To set an environment variable, capitalize the option name, prefix it with *PGBACKREST_*, and replace dashes with underscores.

The following table illustrates how to specify an option using the `pgbackrest.conf` file, command line argument, and environment variable.

| Configuration File        | Command Line Argument | Environment Variable          |
|---------------------------|-----------------------|-------------------------------|
| `pg1-host=1.2.3.4`        | `--pg1-host=1.2.3.4`  | `PGBACKREST_PG1_HOST=1.2.3.4` |
| `compress=y`              | `--compress`          | `PGBACKREST_COMPRESS=y`       |
| `delta=n`                 | `--no-delta`          | `PGBACKREST_DELTA=n`          |

#### Options Precedence

For all options, the order of precedence (highest to lowest) is as follows:

- Command line argument
- Environment variable
- `[stanza:command]`
- `[stanza]`
- `[global:command]`
- `[global]`
- Default (internal)

#### Configuring Multiple Database Hosts in a Stanza

Each stanza could configure multiple database hosts. As such, you can also configure every `pg1-*` configuration option for host 2 with `pg2-*`, for host 3 with `pg3-*`, etc. This assume that those database clusters are linked together (e.g. using _Streaming Replication_).

---

### Glossary

You can find a complete list of commands and configuration keys in the project documentation:

- [Command reference](http://www.pgbackrest.org/command.html) for command-line operations.

- [Configuration reference](http://www.pgbackrest.org/configuration.html) for creating _pgBackRest_ configurations.

---

### Concepts

The following concepts are defined as they are relevant to _pgBackRest_, **PostgreSQL**, and this user guide.

#### Backup

A backup is a consistent copy of a database cluster that can be restored to recover from a hardware failure, perform point-in-time recovery, or to bring up a new standby.

**Full Backup**: _pgBackRest_ copies the entire content of the database cluster to the backup. The first backup of the database cluster is always a full backup. _pgBackRest_ restores a full backup directly. The full backup does not depend on any files outside of the full backup for consistency.

**Differential Backup**: _pgBackRest_ copies only the database cluster files that have changed since the last full backup. To restore a differential backup, _pgBackRest_ copies all of the files in the chosen differential backup and the appropriate unchanged files from the previous full backup. The advantage of a differential backup is that it requires less disk space than a full backup, however, the differential backup and the full backup must be valid to restore the differential backup.

**Incremental Backup**: _pgBackRest_ copies only the database cluster files that have changed since the last backup (which can be an incremental backup, a differential backup, or a full backup). Since an incremental backup only includes the files changed since the prior backup, they are generally much smaller than a full or differential backup. An incremental backup requires other backups to be valid before restoring the incremental backup. Since the incremental backup includes only those files that have been changed or added since the last backup, all prior incremental backups (back to the prior differential and the prior full backup) must be valid to restore the incremental backup. If no differential backup exists, then all prior incremental backups (back to the prior full backup and the full backup itself) must be valid to restore the incremental backup.

#### Restore

A restore is an act of copying a backup to a system where it will be started as a live database cluster. A restore requires the backup files and one or more WAL segments in order to work correctly.

#### Write Ahead Log (WAL)

WAL logging is the mechanism that **PostgreSQL** uses to ensure that no committed changes are lost. Transactions are written sequentially to the WAL and a transaction is considered to be committed when those writes are flushed to disk. Later, a background process writer writes the changes into the main database cluster files (also known as the heap). In the event of a crash, the WAL is replayed to make the database consistent.

WAL is conceptually infinite but in practice is broken up into individual 16MB files called segments. WAL segments follow the naming convention 0000000100000A1E000000FE where the first eight hexadecimal digits represent the timeline and the next 16 digits are the log sequence number (LSN).

#### Encryption

Encryption is the process of converting data into a format that is unrecognizable unless you provide the appropriate password (also referred to as passphrase). To prevent unauthorized access to data stored within the repository, _pgBackRest_ will encrypt the repository after authentication with a user-provided password.
