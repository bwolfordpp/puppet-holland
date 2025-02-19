# Reference
<!-- DO NOT EDIT: This document was generated by Puppet Strings -->

## Table of Contents

**Classes**

_Public Classes_

* [`holland`](#holland): Common setup and resources for the Holland Backup Manager
* [`holland::mongodump`](#hollandmongodump): Manage the mongodump provider for the Holland Backup Manager
* [`holland::mysqldump`](#hollandmysqldump): Manage the Holland mysqldump provider

_Private Classes_

* `holland::config`: Manage the overall [Holland Backup Manager](http://hollandbackup.org/)
configuration.
* `holland::config::remove_default`: An exec to remove the default backupset if it doesn't exist.
* `holland::install`: Basic install of Holland resources
* `holland::mongodump::config`: Configuration for the Holland mongodump provider
* `holland::mongodump::install`: Install the Holland mongodump provider
* `holland::mysqldump::config`: Manage the Holland mysqldump provider configuration.
* `holland::mysqldump::install`: Manage the `holland-mysqldump` package

**Defined types**

* [`holland::mongodump::backupset`](#hollandmongodumpbackupset): Configures a mongodump backup set for Holland
* [`holland::mysqldump::backupset`](#hollandmysqldumpbackupset): Configures a mysqldump backup set for Holland

## Classes

### holland

Common setup and resources for the [Holland Backup Manager](http://hollandbackup.org/).
There isn't an [Augeas](http://augeas.net/) lens for `holland.conf` in the
upstream project yet so we'll need to provide one ourselves to manage the
main configuration from more than one class. Since Puppet requires
`augeas-libs` we don't need to manage the parrent directories since they'll
already be in place.

#### Examples

##### Basic

```puppet
include holland
```

#### Parameters

The following parameters are available in the `holland` class.

##### `ensure`

Data type: `Enum['absent', 'present']`

Should Holland be installed or not.

Default value: present

##### `backup_directory`

Data type: `String`

Top-level directory where backups are held.

Default value: '/var/spool/holland'

##### `logfile`

Data type: `String`

The file Holland logs to

Default value: '/var/log/holland/holland.log'

##### `log_level`

Data type: `Enum[
    'debug',
    'info',
    'warning',
    'error',
    'critical'
  ]`

Sets the verbosity of Holland’s logging process.

Default value: 'info'

##### `path`

Data type: `String`

Defines a path for holland and its spawned processes

Default value: '/usr/local/bin:/usr/local/sbin:/bin:/sbin:/usr/bin:/usr/sbin'

##### `plugin_dirs`

Data type: `String`

Defines where the plugins can be found. This can be a comma-separated list
but usually does not need to be modified.

Default value: '/usr/share/holland/plugins'

##### `umask`

Data type: `String`

Sets the umask of the resulting backup files.

Default value: '0007'

### holland::mongodump

Manage the [mongodump](http://docs.hollandbackup.org/docs/provider_configs/mongodump.html)
provider for the [Holland Backup Manager](https://hollandbackup.org/)

**Note**: This does not install the actual `mongodump` command since there
are several different options for doing so. Especially with software
collections.

#### Examples

##### Basic

```puppet
include holland::mongodump
```

#### Parameters

The following parameters are available in the `holland::mongodump` class.

##### `ensure`

Data type: `Enum['absent', 'present']`

Should the plugin be installed or not.

Default value: present

##### `additional_options`

Data type: `Optional[String]`

Any additional options to the `mongodump` command-line utility these should
show up exactly as they are on the command line. e.g.: `'--gzip'`

Default value: `undef`

##### `authentication_database`

Data type: `String`

The database the mongo user needs to authenticate against.

Default value: ''

##### `compression_level`

Data type: `Integer[0]`

What compression level to use. Lower numbers mean faster compression, though
also generally a worse compression ratio. Generally, levels 1-3 are
considered fairly fast and still offer good compression for textual data.
Levels above 7 can often cause a larger impact on the system due to needing
much more CPU resources. Setting the level to 0 effectively disables
compresion.

Default value: 1

##### `compression_method`

Data type: `Enum[
    'gzip',
    'gzip-rsyncable',
    'bzip2',
    'pbzip2',
    'lzop'
  ]`

Which compression method to use. Note that lzop is not often installed by
default on many Linux distributions and may need to be installed separately.

Default value: 'gzip'

##### `host`

Data type: `String`

Hostname for mongodump to connect with.

Default value: 'localhost'

##### `password`

Data type: `String`

Password for mongodump to authenticate with.

Default value: ''

##### `username`

Data type: `String`

Username for mongodump to authenticate with.

Default value: ''

### holland::mysqldump

This class manages the [Holland Backup Manager](http://hollandbackup.org/)
`mysqldump` provider.

#### Examples

##### Basic

```puppet
include holland::mysqldump
```

#### Parameters

The following parameters are available in the `holland::mysqldump` class.

##### `ensure`

Data type: `Enum['absent', 'present']`

Should the mysqldump provider be installed or not.

Default value: present

##### `additional_options`

Data type: `String`

Specify additional options directly to the `mysqldump` command if there is
no native Holland option for it. These should show up exactly as they would
on the command line. e.g.: `'--flush-privileges --reset-master'`

Default value: ''

##### `bin_log_position`

Data type: `Enum['no', 'yes']`

Record the binary log name and position at the time of the backup. **Note**
that if both `'stop-slave'` and `'bin-log-position'` are enabled, Holland
will grab the master binary log name and position at the time of the backup
which can be useful in using the backup to create slaves or for point in
time recovery using the master’s binary log. This information is found
within the `'backup.conf'` file located in the backup-set destination
directory (`/var/spool/holland/<backup-set>/<backup>` by default).

For example:

    [mysql:replication]
    slave_master_log_pos = 4512
    slave_master_log_file = 260792-mmm-agent1-bin-log.000001

Default value: 'no'

##### `compress_bin_path`

Data type: `Optional[String]`

This only needs to be defined if the compression utility is in a
non-standard location, or not in the system path.

Default value: `undef`

##### `compress_inline`

Data type: `Enum['no', 'yes']`

Whether or not to pipe the output of `mysqldump` into the compression
utility. Enabling this is recommended since it usually only marginally
impacts performance, particularly when using a lower compression level.

Default value: 'yes'

##### `compress_level`

Data type: `Integer[0, 9]`

Specify the compression ratio from `0` to `9`. The lower the number, the
lower the compression ratio, but the faster the backup will take. Generally,
setting the lever to `1` or `2` results in favorable compression of textual
data and is noticeably faster than the higher levels. Setting the level to
`0` effectively disables compression.

Default value: 1

##### `compress_method`

Data type: `Enum['gzip', 'pigz', 'bzip', 'lzop', 'lzma']`

Define which compression method to use.  Note that `lzop` and `lzma` may
not be available on every system and may need to be compiled / installed.

Default value: 'gzip'

##### `databases`

Data type: `Optional[String]`

Comma-delimited glob patterns for matching databases. Only databases
matching these patterns will be backed up. The default is`'*'` which
includes everything.

Default value: `undef`

##### `defaults_extra_file`

Data type: `String`

Comma seperated list of locations to look for the MySQL conection
information using the standard `.my.cnf` conventions.

Default value: '/root/.my.cnf,~/.my.cnf,'

##### `dump_events`

Data type: `Enum['no', 'yes']`

Whether or not to dump events explicitly.  Like routines, events are stored
in the 'mysql' database. Nonetheless, it can sometimes be convenient to
include them in the backup-set directly. **Note**: This feature requires
MySQL 5.1 or later.

Default value: 'no'

##### `dump_routines`

Data type: `Enum['no', 'yes']`

Whether or not to backup routines in the backup set directly. Routines are
stored in the 'mysql' database, but it can sometimes be convenient to
include them in a backup-set directly.

Default value: 'no'

##### `exclude_databases`

Data type: `Optional[String]`

Comma-delimited glob patterns to exclude particular databases.

Default value: `undef`

##### `exclude_tables`

Data type: `Optional[String]`

Comma-delimited glob patterns to exclude particular tables.

Default value: `undef`

##### `file_per_database`

Data type: `Enum['no', 'yes']`

Whether or not to split up each database into its own file.  Note that it
can be more consistent and efficient to backup all databases into one file,
however this means that restore a single database can be difficult if
multiple databases are defined in the backup set.

Default value: 'no'

##### `flush_logs`

Data type: `Enum['no', 'yes']`

Whether or not to run `FLUSH LOGS` in MySQL with the backup.  When
`FLUSH LOGS` is actually executed depends on which if database filtering is
being used and whether or not `file-per-database` is enabled.  Generally
speaking, it does not make sense to use `flush-logs` with
`file-per-database` since the binary logs will not be consistent with the
backup.

Default value: 'no'

##### `lock_method`

Data type: `Enum[
    'flush-lock',
    'lock-tables',
    'single-transaction',
    'auto-detect',
    'none'
  ]`

`flush-lock` will place a global lock on all tables involved in the backup
regardless of whether or not they are in the backup-set. If
`file-per-database` is enabled, then `flush-lock` will lock all tables for
every database being backed up. In other words, this option may not make
much sense when using `file-per-database`.

`lock-tables` will lock all tables involved in the backup. If
`file-per-database` is enabled, then `lock-tables` will only lock all the
tables associated with that database.

`single-transaction` will force running a backup within a transaction.  This
allows backing up of transactional tables without imposing a lock however
will NOT properly backup non-transacitonal tables.

`auto-detect` will choose single-transaction unless Holland finds
non-transactional tables in the backup-set.

`none` will completely disable locking. This is generally only viable on a
MySQL slave and only after traffic has been diverted, or slave services
suspended.

Default value: 'auto-detect'

##### `mysql_binpath`

Data type: `Optional[String]`

Defines the location of the MySQL binary utilities. If not provided,
Holland will use whatever is in the path.

Default value: `undef`

##### `mysql_host`

Data type: `Optional[String]`

The FQDN of the remote host to connect to MySQL on.

Default value: `undef`

##### `mysql_password`

Data type: `Optional[String]`

The password for the MySQL user.

Default value: `undef`

##### `mysql_port`

Data type: `Optional[Integer]`

Used if MySQL is running on a port other than `3306`.

Default value: `undef`

##### `mysql_socket`

Data type: `Optional[String]`

The socket file to connect to MySQL with. eg. `'/tmp/mysqld.sock'`.

Default value: `undef`

##### `mysql_user`

Data type: `Optional[String]`

The user to connect to MySQL as.

Default value: `undef`

##### `stop_slave`

Data type: `Enum['no', 'yes']`

This is useful only when running Holland on a MySQL slave. Instructs
Holland to suspend slave services on the server prior to running the backup.
Suspending the slave does not change the backups, but does prevent the
slave from spooling up relay logs.  The default is not to suspend the slave
(if applicable).

Default value: 'no'

##### `tables`

Data type: `Optional[String]`

Only include the specified tables. Comma seperated glob patterns.

Default value: `undef`

## Defined types

### holland::mongodump::backupset

Configures a mongodump backup set for Holland

#### Examples

##### Inherits from provider

```puppet
include ::holland::mongodump

holland::mongodump::backupset { "localhost":
  ensure                  => present,
  authentication_database => 'admin',
  host                    => 'localhost',
  password                => 'SomeThingToChange',
  username                => 'admin',
}
```

#### Parameters

The following parameters are available in the `holland::mongodump::backupset` defined type.

##### `ensure`

Data type: `Enum['absent', 'present']`

Should the backup set be installed or not.

Default value: present

##### `additional_options`

Data type: `Optional[String]`

Any additional options to the `mongodump` command-line utility these should
show up exactly as they are on the command line. e.g.: `'--gzip'`

Default value: `undef`

##### `authentication_database`

Data type: `Optional[String]`

The database the mongo user needs to authenticate against.

Default value: `undef`

##### `auto_purge_failures`

Data type: `Enum['no', 'yes']`

Specifies whether to keep a failed backup or to automatically remove the
backup directory. By default this is on with the intention that whatever
process is calling holland will retry when a backup fails. This behavior
can be disabled by setting `auto-purge-failures = no` when partial backups
might be useful or when troubleshooting a backup failure.

Default value: 'yes'

##### `backups_to_keep`

Data type: `Integer[1]`

Specifies the number of backups to keep for a backup-set.

Default value: 1

##### `compress_inline`

Data type: `Enum['no', 'yes']`

Whether or not to pipe the output of `mongodump` into the compression
utility. Enabling this is recommended since it usually only marginally
impacts performance, particularly when using a lower compression level.

Default value: 'yes'

##### `compression_level`

Data type: `Integer[0]`

What compression level to use. Lower numbers mean faster compression, though
also generally a worse compression ratio. Generally, levels 1-3 are
considered fairly fast and still offer good compression for textual data.
Levels above 7 can often cause a larger impact on the system due to needing
much more CPU resources. Setting the level to 0 effectively disables
compresion.

Default value: 1

##### `compression_method`

Data type: `Enum[
    'gzip',
    'gzip-rsyncable',
    'bzip2',
    'pbzip2',
    'lzop'
  ]`

Which compression method to use. Note that lzop is not often installed by
default on many Linux distributions and may need to be installed separately.

Default value: 'gzip'

##### `estimated_size_factor`

Data type: `Float`

Specifies the scale factor when Holland decides if there is enough free
space to perform a backup. This number is multiplied against what each
individual plugin reports its estimated backup size when Holland is
verifying sufficient free space for the backupset.

Default value: 1.0

##### `host`

Data type: `Optional[String]`

Hostname for mongodump to connect with.

Default value: `undef`

##### `password`

Data type: `Optional[String]`

Password for mongodump to authenticate with.

Default value: `undef`

##### `purge_policy`

Data type: `Enum[
    'manual',
    'before-backup',
    'after-backup'
  ]`

Specifies when to run the purge routine on a backupset. By default this is
run after a new successful backup completes. Up to `backups_to_keep` backups
will be retained including the most recent.

`before-backup` will run the purge routine just before a new backup starts.
This will retain up to `backups_to_keep` backups before the new backup is
even started allowing purging all previous backups if `backups_to_keep` is
set to `0`. This behavior is useful if some other process is retaining
backups off-server and disk space is at a premium.

`manual` will never run the purge routine automatically. Either
`holland purge` must be run externally or an explicit removal of desired
backup directories can be done at some later time.

Default value: 'after-backup'

##### `username`

Data type: `Optional[String]`

Username for mongodump to authenticate with.

Default value: `undef`

### holland::mysqldump::backupset

Configures a mysqldump backup set for Holland

#### Examples

##### Basic

```puppet
holland::mysqldump::backupset { 'namevar': }
```

#### Parameters

The following parameters are available in the `holland::mysqldump::backupset` defined type.

##### `ensure`

Data type: `Enum['absent', 'file']`

Wheither to ensure the configuration is installed or not.

Default value: file

##### `additional_options`

Data type: `Optional[String]`

Specify additional options directly to the `mysqldump` command if there is
no native Holland option for it. These should show up exactly as they would
on the command line. e.g.: `'--flush-privileges --reset-master'`

Default value: `undef`

##### `auto_purge_failures`

Data type: `Enum['no', 'yes']`

Specifies whether to keep a failed backup or to automatically remove the
backup directory. By default this is on with the intention that whatever
process is calling holland will retry when a backup fails. This behavior
can be disabled by setting `auto-purge-failures = no` when partial backups
might be useful or when troubleshooting a backup failure.

Default value: 'yes'

##### `backups_to_keep`

Data type: `Integer`

Specifies the number of backups to keep for a backup-set.

Default value: 1

##### `compress_bin_path`

Data type: `Optional[String]`

This only needs to be defined if the compression utility is in a
non-standard location, or not in the system path.

Default value: `undef`

##### `compress_inline`

Data type: `Enum['no', 'yes']`

Whether or not to pipe the output of `mysqldump` into the compression
utility. Enabling this is recommended since it usually only marginally
impacts performance, particularly when using a lower compression level.

Default value: 'yes'

##### `compress_level`

Data type: `Integer[0, 9]`

Specify the compression ratio from `0` to `9`. The lower the number, the
lower the compression ratio, but the faster the backup will take. Generally,
setting the lever to `1` or `2` results in favorable compression of textual
data and is noticeably faster than the higher levels. Setting the level to
`0` effectively disables compression.

Default value: 1

##### `compress_method`

Data type: `Enum['gzip', 'pigz', 'bzip', 'lzop', 'lzma']`

Define which compression method to use.  Note that `lzop` and `lzma` may
not be available on every system and may need to be compiled / installed.

Default value: 'gzip'

##### `databases`

Data type: `Optional[String]`

Comma-delimited glob patterns for matching databases. Only databases
matching these patterns will be backed up. The default is`'*'` which
includes everything.

Default value: `undef`

##### `defaults_extra_file`

Data type: `Optional[String]`

Comma seperated list of locations to look for the MySQL conection
information using the standard `.my.cnf` conventions.

Default value: `undef`

##### `dump_events`

Data type: `Optional[Enum['no', 'yes']]`

Whether or not to dump events explicitly.  Like routines, events are stored
in the 'mysql' database. Nonetheless, it can sometimes be convenient to
include them in the backup-set directly. **Note**: This feature requires
MySQL 5.1 or later.

Default value: `undef`

##### `dump_routines`

Data type: `Optional[Enum['no', 'yes']]`

Whether or not to backup routines in the backup set directly. Routines are
stored in the 'mysql' database, but it can sometimes be convenient to
include them in a backup-set directly.

Default value: `undef`

##### `estimated_size_factor`

Data type: `Float`

Specifies the scale factor when Holland decides if there is enough free
space to perform a backup. This number is multiplied against what each
individual plugin reports its estimated backup size when Holland is
verifying sufficient free space for the backupset.

Default value: 1.0

##### `exclude_databases`

Data type: `Optional[String]`

Comma-delimited glob patterns to exclude particular databases.

Default value: `undef`

##### `exclude_tables`

Data type: `Optional[String]`

Comma-delimited glob patterns to exclude particular tables.

Default value: `undef`

##### `file_per_database`

Data type: `Optional[Enum['no', 'yes']]`

Whether or not to split up each database into its own file.  Note that it
can be more consistent and efficient to backup all databases into one file,
however this means that restore a single database can be difficult if
multiple databases are defined in the backup set.

Default value: `undef`

##### `flush_logs`

Data type: `Optional[Enum['no', 'yes']]`

Whether or not to run `FLUSH LOGS` in MySQL with the backup.  When
`FLUSH LOGS` is actually executed depends on which if database filtering is
being used and whether or not `file-per-database` is enabled.  Generally
speaking, it does not make sense to use `flush-logs` with
`file-per-database` since the binary logs will not be consistent with the
backup.

Default value: `undef`

##### `lock_method`

Data type: `Enum[
    'flush-lock',
    'lock-tables',
    'single-transaction',
    'auto-detect',
    'none'
  ]`

`flush-lock` will place a global lock on all tables involved in the backup
regardless of whether or not they are in the backup-set. If
`file-per-database` is enabled, then `flush-lock` will lock all tables for
every database being backed up. In other words, this option may not make
much sense when using `file-per-database`.

`lock-tables` will lock all tables involved in the backup. If
`file-per-database` is enabled, then `lock-tables` will only lock all the
tables associated with that database.

`single-transaction` will force running a backup within a transaction.  This
allows backing up of transactional tables without imposing a lock however
will NOT properly backup non-transacitonal tables.

`auto-detect` will choose single-transaction unless Holland finds
non-transactional tables in the backup-set.

`none` will completely disable locking. This is generally only viable on a
MySQL slave and only after traffic has been diverted, or slave services
suspended.

Default value: 'auto-detect'

##### `mysql_binpath`

Data type: `Optional[String]`

Defines the location of the MySQL binary utilities. If not provided,
Holland will use whatever is in the path.

Default value: `undef`

##### `mysql_host`

Data type: `Optional[String]`

The FQDN of the remote host to connect to MySQL on.

Default value: `undef`

##### `mysql_password`

Data type: `Optional[String]`

The password for the MySQL user.

Default value: `undef`

##### `mysql_port`

Data type: `Optional[Integer]`

Used if MySQL is running on a port other than `3306`.

Default value: `undef`

##### `mysql_socket`

Data type: `Optional[String]`

The socket file to connect to MySQL with. eg. `'/tmp/mysqld.sock'`.

Default value: `undef`

##### `mysql_user`

Data type: `Optional[String]`

The user to connect to MySQL as.

Default value: `undef`

##### `purge_policy`

Data type: `Enum['manual', 'before-backup', 'after-backup']`

Specifies when to run the purge routine on a backupset. By default this is
run after a new successful backup completes. Up to `backups_to_keep` backups
will be retained including the most recent.

`before-backup` will run the purge routine just before a new backup starts.
This will retain up to `backups_to_keep` backups before the new backup is
even started allowing purging all previous backups if `backups_to_keep` is
set to `0`. This behavior is useful if some other process is retaining
backups off-server and disk space is at a premium.

`manual` will never run the purge routine automatically. Either
`holland purge` must be run externally or an explicit removal of desired
backup directories can be done at some later time.

Default value: 'after-backup'

##### `stop_slave`

Data type: `Optional[Enum['no', 'yes']]`

This is useful only when running Holland on a MySQL slave. Instructs
Holland to suspend slave services on the server prior to running the backup.
Suspending the slave does not change the backups, but does prevent the
slave from spooling up relay logs.  The default is not to suspend the slave
(if applicable).

Default value: `undef`

##### `tables`

Data type: `Optional[String]`

Only include the specified tables. Comma seperated glob patterns.

Default value: `undef`

