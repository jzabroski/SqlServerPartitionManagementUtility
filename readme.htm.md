# Partition Management Utility v 3.0

Stuart Ozer, Microsoft Corporation

[http://sqlpartitionmgmt.codeplex.com/](http://sqlpartitionmgmt.codeplex.com/)

December 2012

Provided AS-IS, with no warranties expressed or implied

Licensed via [Microsoft Public License](http://sqlpartitionmgmt.codeplex.com/license) http://sqlpartitionmgmt.codeplex.com/license

## Executable name
ManagePartition.exe

This 3.0 release of the Partition Management Utility supports SQL Server 2012, 2008R2, 2008 and 2005.

New features include: 
* Support for <span class=SpellE>Columnstore</span> Indexes in SQL Server 2012
* Ability to generate TSQL Scripts for all utility operations, in addition to or in lieu of executing operations on the server
* Full support for diverse global locales and their date/time representations
* The utility requires **.Net Framework version 3.5**, along with the following DLLs running on the client or server that executes the utility:
  * SQL Server 2012 System CLR Types
    * [X86 Package (SQLSysClrTypes.msi)](http://go.microsoft.com/fwlink/?LinkID=239643&amp;clcid=0x409)
    * [X64 Package (SQLSysClrTypes.msi)](http://go.microsoft.com/fwlink/?LinkID=239644&amp;clcid=0x409)
  * SQL Server 2012 Management Objects (SMO)
    * [X86 Package (SharedManagementObjects.msi)](http://go.microsoft.com/fwlink/?LinkID=239658&amp;clcid=0x409)
    * [X64 Package (SharedManagementObjects.msi)](http://go.microsoft.com/fwlink/?LinkID=239659&amp;clcid=0x409)

Description:
The general problems addressed by this utility are:
1. Guaranteeing that the staging table required to SWITCH in or out a partition is always
created correctly **just in time**, with the right indexes, columns, foreign
keys, indexed views, and partition-specific check constraint in the `filegroup` 
corresponding to the target partition of a partitioned table.
2. Ensuring that any partition management
scripts stay in synch with the possible index changes, column changes or
foreign key constraints of partition tables over time.
    Without this utility, any change to a
    partition tables DDL would require an equivalent change in a TSQL-based
    partition management script, along with associated testing, etc.
3. Providing an easy mechanism to quickly empty
a selected partition from a partitioned table with a single command-line
4. Allowing fine-tuned partition data loading 
including scenarios in which you want to create a non-indexed staging table (or
a table with only a Clustered index) and populate it, and create necessary
additional indexes later prior to a SWITCH in operation.

The utility must be run from the command line or within an SSIS package.

Command line parameters determine behavior.

You may perform one of five different functions depending on the COMMAND parameter.

Additional parameters define the connection
(server, integrated security, etc); database, schema, partitioned table name,
etc.

You have the option of identifying
a partition to manage either by explicit partition number (/p:)
OR by specifying a string representation of a value that can be input to the
partition range function to determine a partition number (/v:).

COMMAND Description:
5 mutually-exclusive commands are supported as the /C: parameter to the
executable.

| Command | Description |
| ------- | ----------- |
| ClearPartition | Empties a partition of all rows by creating a staging table and calling SWITCH. There is the option to keep or drop the
staging table (/K), and the name of the staging table is optional (/A) |
| CreateStagingFull | Creates a staging table to match a selected partition of a partition table, including all indexes, fkeys, , check constraint, indexed views,
etc. This can then be populated for a later SWITCH in to the partition table. |
| CreateStagingNoindex | Same as `CreateStagingFull` except it does not create the indexes or
indexed views. Use this command if you
want to populate the staging table through a bulk-load, SSIS or insert-select
while indexes are not present for performance reasons. After the table is populated, you must rerun
the program using the `IndexStaging` command in order for
the table to be capable of a SWITCH operation. |
| CreateStagingClusteredIndex | Same as `CreateStagingNoIndex` except it also creates the Clustered Index
on the table, deferring all `nonclustered` indexes and
indexed views until the program is run again with the `IndexStaging` command. Use this command if you want to populate the staging table through a
bulk-load, SSIS or insert-select while only the Clustered index is in place. |
| IndexStaging | Builds any indexes
or indexed views on a staging table to match those of the associated partition
table, unless they are already present.
Use this if you previously used the `CreateStagingNoindex` or `CreateStagingClusteredIndex` command to create the
table. After this command the table will be ready for a SWITCH operation. |

Note
that if you are using `ColumnStore` indexes in SQL
Server 2012 and switching IN to a partitioned table, you must populate your
staging tables before the `ColumnStore` index is built.
So you will first want to create the staging
tables using the `CreateStagingNoindex` or `CreateStagingClusteredIndex` command, then load the data, then
rerun the utility using `IndexStaging` command.

Usage:
ManagePartition

Command
CreateStagingFull
Server:<string>
/S
Server name, default = (local)

User:<string>
/U
User name, default = null

Password:<string>
/P
Password, default = null

Integrated[+|-]
Integrated Security, default +

Database:<string>
Database name

Schema:<string>
Schema name - use quotes as delimiter if needed

PartitionTable:<string>
Partition table name - use quotes as delimiter if needed

PartitionNumber:<int>
Partition Number

PartitionRangeValue:<string>
/v Value string to input to partition function to specify partition number

StagingTable:<string>
/A Staging Table Name, default = null

ScriptOption:<char>
/O Scripting Option -- i Include Script, o Script Only (no execute)

ScriptFile:<string>
/f path and name of file for
generated TSQL -- if excluded, script any output to stdout

Keep[+|-]
/K Keep Staging Table
if ClearPartition, default +

<Command> = =ClearPartition
| CreateStagingFull | CreateStagingNoindex
| CreateStagingClusteredIndex | IndexStaging

Examples:
```
ManagePartition /C:CreateStagingFull d:sample s:dbo /t:Address /p:4 /A:myStagingTbl 
```

```
ManagePartition /C:CreateStagingFull /d:sample /s:dbo /t:Address /v:2009-01-01 /A:myStagingTbl /f:c:\scripts\myscript.sql /O:i
```

```
ManagePartition /C:ClearPartition /d:sample /s:dbo /t:Address /p:1 /K- 
```

Return
Codes:
| Code | Description |
| ---- | ----------- |
| 0 | normal exit |
| 1 | invalid or missing parameters  |
| 2 | exception encountered |
