# Sqoop

sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities
----------------------------------------------------------------------
import the table cities into the directory /etl/input/cities:
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--target-dir /etl/input/cities
----------------------------------------------------------------------
To specify the parent directory for all your Sqoop jobs, instead use the --warehousedir
parameter:
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--warehouse-dir /etl/input/
-----------------------------------------------------------------------
Importing Only a Subset of Data
to import only USA cities from the table cities, you can issue the following Sqoop command:
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--where "country = 'USA'"
---------------------------------------------------------------------
Protecting Your Password
You have two options besides specifying the password on the command line with the
--password parameter. The first option is to use the parameter -P that will instruct
Sqoop to read the password from standard input. Alternatively, you can save your password
in a file and specify the path to this file with the parameter --password-file.
Here’s a Sqoop execution that will read the password from standard input:
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--table cities \
-P
Here’s an example of reading the password from a file:
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--table cities \
--password-file my-sqoop-password
-------------------------------------------------------------------------------------------------------
Using a File Format Other Than CSV
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--as-sequencefile

sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--as-avrodatafile
----------------------------------------------------------------------------------------------------
Compressing Imported Data
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--table cities \
--compress

By default, when using the --compress parameter, output files
will be compressed using the GZip codec, and all files will end up with a .gz extension.
You can choose any other codec using the --compression-codec parameter. The following
example uses the BZip2 codec instead of GZip (files on HDFS will end up having
the .bz2 extension):
sqoop import --compress \
--compression-codec org.apache.hadoop.io.compress.BZip2Codec

Another benefit of leveraging MapReduce’s compression abilities is that Sqoop can make
use of all Hadoop compression codecs out of the box. You don’t need to enable compression
codes within Sqoop itself. That said, Sqoop can’t use any compression algorithm
not known to Hadoop. Prior to using it with Sqoop, make sure your desired codec
is properly installed and configured across all nodes in your cluster.

Compression codecs
Splittable		 Not Splittable
BZip2, LZO 	GZip, Snappy

-----------------------------------------------------------------------------------------------
Speeding Up Transfers
 sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--table cities \
--direct

Rather than using the JDBC interface for transferring data, the direct mode delegates
the job of transferring data to the native utilities provided by the database vendor. In
the case of MySQL, the mysqldump and mysqlimport will be used for retrieving data
from the database server or moving data back.

limitation of the direct mode is that not all parameters are supported. As the
native utilities usually produce text output, binary formats like SequenceFile or Avro
won’t work.

------------------------------------------------------------------------------------------------
Controlling Parallelism
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities -m 10

By default mapper size is 4
if you’re transferring only 4 rows yet set --num-mappers to 10 mappers, only 4 mappers will be used, 
as the other 6 mappers would not have any data to transfer.

------------------------------------------------------------------------------------------------------------
Encoding NULL Values
You can override the NULL substitution string with the --null-string and --null-nonstring
parameters to any arbitrary value. For example, use the following command to
override it to \N:
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--null-string '\\N' \
--null-non-string '\\N'
------------------------------------------------------------------------------------------------------------
Importing All Your Tables
sqoop import-all-tables \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop

if you need to import all tables from the database except cities and countries, you would use the 
following command:
sqoop import-all-tables \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--exclude-tables cities,countries
-----------------------------------------------------------------------------------------------------------
Importing Only New Data

You have a database table with an INTEGER primary key. You are only appending new
rows, and you need to periodically sync the table’s state to Hadoop for further processing.

Activate Sqoop’s incremental feature by specifying the --incremental parameter. The
parameter’s value will be the type of incremental import. When your table is only getting
new rows and the existing ones are not changed, use the append mode.
Incremental import also requires two additional parameters: --check-column indicates
a column name that should be checked for newly appended data, and --last-value
contains the last value that successfully imported into Hadoop.
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table visits \
--incremental append \
--check-column id \
--last-value 1

---------------------------------------------------------------------------------------------------------
Incrementally Importing Mutable Data
Use the lastmodified mode instead of the append mode.

sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table visits \
--incremental lastmodified \
--check-column last_update_date \
--last-value "2013-05-22 01:01:01"

--------------------------------------------------------------------------------------------------------
Preserving the Last Imported Value
You can take advantage of the built-in Sqoop metastore that allows you to save all parameters
for later reuse. You can create a simple incremental import job with the following
command:

sqoop job \
--create visits \
-- \
import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table visits \
--incremental append \
--check-column id \
--last-value 0

And start it with the --exec parameter:
sqoop job --exec visits

list all retained jobs using
sqoop job --list

remove the old job definitions that are no longer needed
sqoop job --delete visits

view content of the saved job definitions
sqoop job --show visits

-----------------------------------------------------------------------------------------------------------
Storing Passwords in the Metastore
set the property sqoop.metastore.client.record.password in the sqoop-site.xml to true:
<configuration>
...
<property>
<name>sqoop.metastore.client.record.password</name>
<value>true</value>
</property>
</configuration>

storing the password inside the metastore is less secure. The
metastore is unencrypted, and thus anyone can easily retrieve your saved password.

----------------------------------------------------------------------------------------------------------
Importing Data from Two Tables

sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--query 'SELECT normcities.id, \
countries.country, \
normcities.city \
FROM normcities \
JOIN countries USING(country_id) \
WHERE $CONDITIONS' \
--split-by id \
--target-dir cities

In addition to the --query parameter, you need to specify the --split-by parameter with the column that should be used for slicing
your data into multiple parallel tasks. This parameter usually automatically defaults to
the primary key of the main table.
To help Sqoop split your query into multiple chunks that can be transferred in parallel, you
need to include the $CONDITIONS placeholder in the where clause of your query. Sqoop
will automatically substitute this placeholder with the generated conditions specifying
which slice of data should be transferred by each individual task. While you could skip
$CONDITIONS by forcing Sqoop to run only one job using the --num-mappers 1 parameter,
such a limitation would have a severe performance impact.

===========================================================================================================
Sqoop Export:

sqoop export \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--export-dir cities

---------------------------------------------------------------------------------------------
Inserting Data in Batches
While Sqoop’s export feature fits your needs, it’s too slow. It seems that each row is inserted in a separate insert statement.

Sqoop offers multiple options for inserting more than one row at a time.
sqoop export \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--export-dir cities \
--batch

The second option is to use the property sqoop.export.records.per.statement to specify the number of records 
that will be used in each insert statement: (specifying multiple rows inside one single insert statement.)
sqoop export \
-Dsqoop.export.records.per.statement=10 \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--export-dir cities

you can set how many rows will be inserted per transaction with the sqoop.export.statements.per.transaction property:
sqoop export \
-Dsqoop.export.statements.per.transaction=10 \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--export-dir cities

The value specified in sqoop.export.statements.per.transaction determines how many insert statements will be
issued on the database prior to committing the transaction and starting a new one.

---------------------------------------------------------------------------------------------------------------------------------
use a staging table to first load data to a temporary table before making changes to the real table.
sqoop export \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--staging-table staging_cities
--------------------------------------------------------------------------------------------------------------------------------
Encoding the NULL Value Differently
sqoop export \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--input-null-string '\\N' \
--input-null-non-string '\\N'

---------------------------------------------------------------------------------------------------------------------------------
Exporting into a Subset of Columns
the corresponding table in your database has more columns than the HDFS data.
* use the --columns parameter to specify which columns (and in what order) are present in the Hadoop data.
sqoop export \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--columns country,city

---------------------------------------------------------------------------------------------------------------------------------
Using Stored Procedures
database already has a workflow for ingesting new data that heavily uses stored procedures instead of direct INSERT statements.
*Instead of using the --table parameter to specify the target table, use the --call parameter followed 
by the name of the stored procedure that should be called
sqoop export \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--call populate_cities

-----------------------------------------------------------------------------------------------------------------------------
Updating an Existing Data Set
You previously exported data from Hadoop, after which you ran additional processing
that changed it. Instead of wiping out the existing data from the database, you prefer to
just update any changed rows.
sqoop export \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--update-key id

-----------------------------------------------------------------------------------------------------------------------------
Updating or Inserting at the Same Time
You have data in your database from a previous export, but now you need to propagate
updates from Hadoop. Unfortunately, you can’t use the update mode, as you have a
considerable number of new rows and you need to export them as well.
sqoop export \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--update-key id \
--update-mode allowinsert
If you need both updates and inserts in the same job, you can activate the so-called
upsert mode with the --update-mode allowinsert parameter. For example:

-----------------------------------------------------------------------------------------------------------------------------
Importing Data Directly into Hive
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--hive-import

If the table in Hive does not exist yet, Sqoop
will simply create it based on the metadata fetched for your table or query. If the table
already exists, Sqoop will import data into the existing table. If you’re creating a new
Hive table, Sqoop will convert the data types of each column from your source table to a type compatible with Hive.

Sometimes the default mapping doesn’t work correctly for your needs; in those cases,
you can use the parameter --map-column-hive to override it.
This parameter expects a comma-separated list of key-value pairs separated by the equal sign (=) in order to
specify which column should be matched to which type in Hive. For example, if you
want to change the Hive type of column id to STRING and column price to DECIMAL,
you can specify the following Sqoop parameters:
sqoop import \
...
--hive-import \
--map-column-hive id=STRING,price=DECIMAL

By using the parameter --hive-overwrite, which will instruct Sqoop to truncate an existing Hive table and load only the newly imported one.

-------------------------------------------------------------------------------------------------------------------------------------
Using Partitioned Hive Tables
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--hive-import \
--hive-partition-key day \
--hive-partition-value "2013-05-22"

Each partition operation must contain the name and value of the partition.
Sqoop can’t use your data to determine which partition this should go into. Instead
Sqoop relies on the user to specify the parameter --hive-partition-value with an appropriate value.

------------------------------------------------------------------------------------------------------------------------------------
Replacing Special Delimiters During Hive Import

-----------------------------------------------------------------------------------------------------------------------------------
Using the Correct NULL String in Hive
sqoop import \
--connect jdbc:mysql://quickstart:3306/sqoop \
--username sqoop \
--password sqoop \
--table cities \
--hive-import \
--null-string '\\N' \
--null-non-string '\\N'
-------------------------------------------------------------------------------------------------------------------------------
Note:
 In entire sqoop process, "No Reducer Involvement" is there only mapper will work.

By default sqoop writes comma (,) delimiter between fields.

To change the delimiter use following option:-
   --fields-terminated-by  '\t'

If table is not having primary key, the number of mappers should be 1, because when multiple mappers are initiated, 
  first mapper automatically points to begining record of the first split and remaining mappers can not point to begining
  of their splits randomly. bcause there is no primary key.

  So, only sequential reading is allowed from beginning of table to the end of the table.

  To make start to end as one split, only one mapper should be intiated.

sqoop import --connect jdbc:mysql://quickstart:3306/practice --username root --password cloudera --table samp -m 1 
--columns name,sal,sal*0.1 --target-dir sqimp7
In --columns option, we can not give  any arithmetic expressions over the column. As, it can not generate new fields
at the time of importing.

For where clause any number of conditions can be given, but $CONDITIONS is mandatory.

 $CONDITIONS is boolean variable with dafault value FALSE.

 when all given expressions are valid, then $CONDITIONS will turn to TRUE.

 If it is true, then sqoop submits statement to rdbms and then importing will be started.

sqoop import --connect jdbc:mysql://quickstart:3306/practice --username root --password cloudera 
--query 'select * from samp where $CONDITIONS' -m 1 --target-dir sqimp8

--Generating new fields at the time of importing:-

sqoop import --connect jdbc:mysql://quickstart:3306/practice --username root --password cloudera --query 'select name,sal,sal*0.1,sal*0.2,sal+(sal*0.2)-(sal*0.1) from samp where $CONDITIONS' -m 1 --target-dir sqimp10

hadoop fs -cat sqimp10/part-m-00000

**Note:-  --table  and --query are mutually exclusive.
