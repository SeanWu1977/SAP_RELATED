###### logon isql

`isql -Usapsa -S<SID> -X -P<PWD>`



###### 1588316 - SYB: Configure automatic database and log backups

`use master`
`exec sp_config_dump @config_name='QMDDB', @stripe_dir = '/sybase/QMD/backups' , @compression = '101' , @verify = 'header'`
`go`
`use master`
`sp_config_dump @config_name='QMDLOG', @stripe_dir = '/sybase/QMD/log_archives', @compression = '101' , @verify = 'header'`

##### check config

`sp_config_dump QMDLOG`

##### backup 

`dump database master using config = 'QMDDB'`
`dump database QMD using config = 'QMDDB'`
`Dumps database QMD using dump configuration QMDDB`
`dump transaction QMD using config = 'QMDLOG'`

> In order to minimize the potential for transaction loss in the case of a disk
> error, we recommend that you schedule a DUMP TRANSACTION command frequently in
> a production environment.



##### 1585981 - SYB: Ensuring Recoverability for SAP ASE

> Perform daily database backups of the master and the <DBSID> database (optional: sybmgmtdb, saptools ; sybsystemprocs; see below for details).
> Dump the transaction log of the <DBSID> database at least once an hour.
> Save a copy of the transaction log dumps to a different medium:
> If you dumped the transaction log to disk, copy these disk dumps to tape afterwards.
> If you dumped the transaction log to tape, clone the tape.
> Retain all database backups plus all transaction log dumps for 28 days.
> Perform a full database recovery test at least once during your backup retention period (that is, restore the database and roll forward through the transaction logs)!



Ensure that databases can be restored from database copies.

Create copies of the following data at regular intervals:
DUMP images of the following databases:

> 1. <DBSID> database (SAP database)
> 2. master database
> 3. sybsystemprocs database (a restore is required when automatic database expansion has been set up)
> 4. sybmgmtdb database (optional)
> 5. saptools database (optional)



It is mandatory that you ensure that the archived log sequence remains unbroken, if the SAP database <DBSID> is used in a production environment. To ensure a complete log sequence, set the following database options:

`'trunc log on chkpt', 'false'`
`'full logging for all', 'true'`
`'enforce dump tran sequence', 'true'`



Set options 'trunc log on chkpt' and 'full logging for all', then dump the <DBSID> database, and set 'enforce dump tran sequence'. Database option 'enforce dump tran sequence' can only be set after a successful 'DUMP DATABASE' command has been performed, and before any changes have been made to the database after the dump.

Set these options using the stored procedure 'sp_dboption' to change database options. (Syntax : sp_dboption [dbname, optname, {true | false}] ). You must change to the 'master' database to be able to change options for a database.

!Careful! : After you have changed these options, inactive log entries will no longer be purged from the transaction log at checkpoints. In order to prevent the transaction log from filling up, you have to get regular dumps of the transaction log.



##### 1801984 - SYB: Automated management of long running transactions

In addition to setting up a scheduled DUMP TRANSACTION command,
it is also strongly recommended that you set up a threshold action that triggers a DUMP TRANSACTION command
 whenever a threshold (fill level) in the log segment has been reached.
