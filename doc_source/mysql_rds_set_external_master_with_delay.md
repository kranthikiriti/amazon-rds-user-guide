# mysql\.rds\_set\_external\_master\_with\_delay<a name="mysql_rds_set_external_master_with_delay"></a>

Configures an Amazon RDS MySQL DB instance to be a Read Replica of an instance of MySQL running external to Amazon RDS and configures delayed replication\.

## Syntax<a name="mysql_rds_set_external_master_with_delay-syntax"></a>

```
CALL mysql.rds_set_external_master_with_delay (
  host_name
  , host_port
  , replication_user_name
  , replication_user_password
  , mysql_binary_log_file_name
  , mysql_binary_log_file_location
  , ssl_encryption
  , delay
);
```

## Parameters<a name="mysql_rds_set_external_master_with_delay-parameters"></a>

 *host\_name*   
The host name or IP address of the MySQL instance running external to Amazon RDS that will become the replication master\.

 *host\_port*   
The port used by the MySQL instance running external to Amazon RDS to be configured as the replication master\. If your network configuration includes SSH port replication that converts the port number, specify the port number that is exposed by SSH\.

 *replication\_user\_name*   
The ID of a user with REPLICATION CLIENT and REPLICATION SLAVE permissions on the MySQL instance running external to Amazon RDS\. We recommend that you provide an account that is used solely for replication with the external instance\.

 *replication\_user\_password*   
The password of the user ID specified in `replication_user_name`\.

 *mysql\_binary\_log\_file\_name*   
The name of the binary log on the replication master contains the replication information\.

 *mysql\_binary\_log\_file\_location*   
The location in the `mysql_binary_log_file_name` binary log at which replication will start reading the replication information\.

 *ssl\_encryption*   
This option is not currently implemented\.  The default is 0\.

 *delay*   
The minimum number of seconds to delay replication from the master\.  
The limit for this parameter is one day \(86400 seconds\)\.

## Usage Notes<a name="mysql_rds_set_external_master_with_delay-usage-notes"></a>

 The master user must run the `mysql.rds_set_external_master_with_delay` procedure\. This procedure must be run on the MySQL DB instance to be configured as the Read Replica of a MySQL instance running external to Amazon RDS\. 

 Before you run `mysql.rds_set_external_master_with_delay`, you must configure the instance of MySQL running external to Amazon RDS to be a replication master\. To connect to the MySQL instance running external to Amazon RDS, you must specify values for `replication_user_name` and `replication_user_password`\. These values must indicate a replication user that has `REPLICATION CLIENT` and `REPLICATION SLAVE` permissions on the external instance of MySQL\. 

**To configure an external instance of MySQL as a replication master**

1. Using the MySQL client of your choice, connect to the external instance of MySQL and create a user account to be used for replication\. The following is an example\.

   ```
   CREATE USER 'repl_user'@'mydomain.com' IDENTIFIED BY 'SomePassW0rd'
   ```

1. On the external instance of MySQL, grant `REPLICATION CLIENT` and `REPLICATION SLAVE` privileges to your replication user\. The following example grants `REPLICATION CLIENT` and `REPLICATION SLAVE` privileges on all databases for the `'repl_user'` user for your domain\.

   ```
   GRANT REPLICATION CLIENT, REPLICATION SLAVE ON *.* TO 'repl_user'@'mydomain.com' 
   IDENTIFIED BY 'SomePassW0rd'
   ```

For more information, see [Replication with a MySQL or MariaDB Instance Running External to Amazon RDS](MySQL.Procedural.Importing.External.Repl.md)\.

**Note**  
We recommend that you use Read Replicas to manage replication between two Amazon RDS DB instances when possible\. When you do so, we recommend that you use only this and other replication\-related stored procedures\. These practices enable more complex replication topologies between Amazon RDS DB instances\. We offer these stored procedures primarily to enable replication with MySQL instances running external to Amazon RDS\. For information about managing replication between Amazon RDS DB instances, see [Working with Read Replicas](USER_ReadRepl.md)\.

After calling `mysql.rds_set_external_master_with_delay` to configure an Amazon RDS DB instance as a Read Replica, you can call [mysql\.rds\_start\_replication](mysql_rds_start_replication.md) on the Read Replica to start the replication process\. You can call [mysql\.rds\_reset\_external\_master](mysql_rds_reset_external_master.md) to remove the Read Replica configuration\.

When you call `mysql.rds_set_external_master_with_delay`, Amazon RDS records the time, the user, and an action of `"set master"` in the `mysql.rds_history` and `mysql.rds_replication_status` tables\.

For disaster recovery, you can use this procedure with the [mysql\.rds\_start\_replication\_until](mysql_rds_start_replication_until.md) or [mysql\.rds\_start\_replication\_until\_gtid](mysql_rds_start_replication_until_gtid.md) stored procedure\. To roll forward changes to a delayed Read Replica to the time just before a disaster, you can run the `mysql.rds_set_external_master_with_delay` procedure\. After the `mysql.rds_start_replication_until` procedure stops replication, you can promote the Read Replica to be the new master DB instance by using the instructions in [Promoting a Read Replica to Be a Standalone DB Instance](USER_ReadRepl.md#USER_ReadRepl.Promote)\. 

To use the `mysql.rds_rds_start_replication_until_gtid` procedure, GTID\-based replication must be enabled\. To skip a specific GTID\-based transaction that is known to cause disaster, you can use the [mysql\.rds\_skip\_transaction\_with\_gtid](mysql_rds_skip_transaction_with_gtid.md) stored procedure\. For more information about working with GTID\-based replication, see [Using GTID\-Based Replication for Amazon RDS MySQL](mysql-replication-gtid.md)\.

The `mysql.rds_set_external_master_with_delay` procedure is available in these versions of Amazon RDS MySQL:
+ MySQL 5\.6\.40 and later 5\.6 versions
+ MySQL 5\.7\.22 and later 5\.7 versions

## Examples<a name="mysql_rds_set_external_master_with_delay-examples"></a>

When run on a MySQL DB instance, the following example configures the DB instance to be a Read Replica of an instance of MySQL running external to Amazon RDS\. It sets the minimum replication delay to one hour \(3,600 seconds\) on the MySQL DB instance\. A change from the MySQL master running external to Amazon RDS is not applied on the MySQL DB instance Read Replica for at least one hour\.

```
call mysql.rds_set_external_master_with_delay(
  'Externaldb.some.com',
  3306,
  'repl_user'@'mydomain.com',
  'SomePassW0rd',
  'mysql-bin-changelog.000777',
  120,
  0,
  3600);
```