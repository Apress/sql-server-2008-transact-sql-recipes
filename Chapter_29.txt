-----------------------------------------------------------------------
-- Source Code: SQL Server 2008 Transact-SQL Recipes, Joseph Sack
-----------------------------------------------------------------------
-- Do not execute the following code in a single batch.  These samples
-- are provided in order to follow along with specific recipes.  
-----------------------------------------------------------------------

-- Performing a Basic Full Backup

USE master
GO
IF NOT EXISTS (SELECT name
FROM sys.databases
WHERE name = 'TestDB')
BEGIN
CREATE DATABASE TestDB
END
GO

USE TestDB
GO

SELECT *
INTO dbo.SalesOrderDetail
FROM AdventureWorks.Sales.SalesOrderDetail
GO

SELECT *
INTO dbo.SalesOrderHeader
FROM AdventureWorks.Sales.SalesOrderHeader
GO

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Oct_14_2008_1617.BAK'

-- CompressingYour Backups

USE master
GO
EXEC sp_configure 'backup compression default', '1'
RECONFIGURE WITH OVERRIDE
GO

SELECT description,value_in_use
FROM sys.configurations
WHERE name = 'backup compression default'

BACKUP DATABASE AdventureWorks
TO DISK = 'C:\Apress\AW_compressed.bak'

BACKUP DATABASE AdventureWorks
TO DISK = 'C:\Apress\AW_uncompressed.bak'
WITH NO_COMPRESSION

SELECT TOP 2 database_name, backup_size, compressed_backup_size
FROM msdb..backupset
ORDER BY backup_finish_date DESC

-- Naming and DescribingYour Backups andMedia

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB.bak'
WITH DESCRIPTION = 'My second recipe backup, TestDB',
NAME = 'TestDB Backup October 14th',
MEDIADESCRIPTION = 'Backups for October 2008, Week 2',
MEDIANAME = 'TestDB_October_2008_Week2'

-- Configuring Backup Retention

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Oct.bak'
WITH RETAINDAYS = 30

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Oct.bak'
WITH INIT

-- Striping Backup Sets

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Stripe1.bak',
DISK = 'D:\Apress\Recipes\TestDB_Stripe2.bak',
DISK = 'E:\Apress\Recipes\TestDB_Stripe3.bak'

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Stripe1.bak'

-- Using a Named Backup Device

USE master
GO
EXEC sp_addumpdevice 'disk', 'TestDBBackup', 'C:\Apress\Recipes\TestDB_Device.bak'

EXEC sp_helpdevice 'TestDBBackup'

BACKUP DATABASE TestDB
TO TestDBBackup

EXEC sp_dropdevice 'TestDBBackup'

-- Mirroring Backup Sets
BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Original.bak'
MIRROR TO DISK = 'D:\Apress\Recipes\TestDB_Mirror_1.bak'
MIRROR TO DISK = 'E:\Apress\Recipes\TestDB_Mirror_2.bak'
MIRROR TO DISK = 'F:\Apress\Recipes\TestDB_Mirror_3.bak'
WITH FORMAT

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Stripe_1_Original.bak',
DISK = 'D:\Apress\Recipes\TestDB_Stripe_2_Original.bak'
MIRROR TO DISK = 'E:\Apress\Recipes\TestDB_Stripe_1_Mirror_1.bak',
DISK = 'F:\Apress\Recipes\TestDB_Stripe_2_Mirror_1.bak'
WITH FORMAT

-- Performing a Transaction Log Backup

BACKUP LOG TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Oct_14_2008_1819.trn'

BACKUP LOG TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Oct_14_2008_1820_Emergency.trn'
WITH NO_TRUNCATE

-- Create Backups Without Breaking the Backup Sequence

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Copy.bak'
WITH COPY_ONLY

BACKUP LOG TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Copy.trn'
WITH COPY_ONLY

-- Performing a Differential Backup

BACKUP DATABASE TestDB
TO DISK = N'C:\Apress\Recipes\TestDB.diff'
WITH DIFFERENTIAL, NOINIT, STATS = 25

-- Backing Up Individual Files or Filegroups

USE master
GO

CREATE DATABASE VLTestDB
ON PRIMARY
( NAME = N'VLTestDB',
FILENAME =
N'c:\Apress\Recipes\VLTestDB.mdf' ,
SIZE = 3048KB ,
FILEGROWTH = 1024KB ),
FILEGROUP FG2
( NAME = N'VLTestDB2',
FILENAME =
N'c:\Apress\Recipes\VLTestDB2.ndf' ,
SIZE = 3048KB ,
FILEGROWTH = 1024KB ),
( NAME = N'VLTestDB3',
FILENAME =
N'c:\Apress\Recipes\VLTestDB3.ndf' ,
SIZE = 3048KB ,
FILEGROWTH = 1024KB )
LOG ON
( NAME = N'VLTestDB_log',
FILENAME =
N'c:\Apress\Recipes\VLTestDB_log.ldf' ,
SIZE = 1024KB ,
FILEGROWTH = 10%)
GO

BACKUP DATABASE VLTestDB
FILEGROUP = 'FG2'
TO DISK = 'C:\Apress\Recipes\VLTestDB_FG2.bak'

USE VLTestDB
GO
EXEC sp_helpfile

BACKUP DATABASE VLTestDB
FILE = 'VLTestDB2',
FILE = 'VLTestDB3'
TO DISK = 'C:\apress\Recipes\VLTestDB_DB2_DB3.bak'

-- Performing a Partial Backup

USE master
GO
ALTER DATABASE VLTestDB
MODIFY FILEGROUP FG2 READONLY
GO

BACKUP DATABASE VLTestDB
READ_WRITE_FILEGROUPS
TO DISK = 'C:\Apress\Recipes\TestDB_Partial_include_FG3.bak'

-- Viewing Backup Metadata

RESTORE LABELONLY
FROM DISK = 'C:\apress\Recipes\TestDB.bak'

RESTORE HEADERONLY
FROM DISK = 'C:\Apress\Recipes\TestDB.bak'

RESTORE FILELISTONLY
FROM DISK = 'C:\Apress\Recipes\TestDB.bak'

RESTORE VERIFYONLY
FROM DISK = 'C:\Apress\Recipes\TestDB.bak'
WITH FILE = 1,
LOADHISTORY

-- Restoring a Database froma Full Backup

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Oct_15_2008.BAK'
GO

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_Oct_15_2008.BAK'
GO

USE master
GO
RESTORE DATABASE TestDB
FROM DISK = 'C:\Apress\Recipes\TestDB_Oct_15_2008.bak'
WITH FILE = 2, REPLACE

USE master
GO
RESTORE DATABASE TrainingDB1
FROM DISK = 'C:\Apress\Recipes\TestDB_Oct_15_2008.BAK'
WITH FILE = 2,
MOVE 'TestDB' TO 'C:\Apress\Recipes\TrainingDB1.mdf',
MOVE 'TestDB_log' TO 'C:\Apress\Recipes\TrainingDB1_log.LDF'

USE master
GO
RESTORE DATABASE TestDB
FROM DISK = 'C:\Apress\Recipes\TestDB_Stripe1.bak',
DISK = 'D:\Apress\Recipes\TestDB_Stripe2.bak',
DISK = 'E:\Apress\Recipes\TestDB_Stripe3.bak'
WITH FILE = 1, REPLACE

-- Restoring a Database froma Transaction Log Backup

IF NOT EXISTS (SELECT name
FROM sys.databases
WHERE name = 'TrainingDB')
BEGIN
CREATE DATABASE TrainingDB
END
GO

-- Add a table and some data to it
USE TrainingDB
GO
SELECT *
INTO dbo.SalesOrderDetail
FROM AdventureWorks.Sales.SalesOrderDetail
GO

BACKUP DATABASE TrainingDB
TO DISK = 'C:\Apress\Recipes\TrainingDB.bak'
GO

BACKUP LOG TrainingDB
TO DISK = 'C:\Apress\Recipes\TrainingDB_Oct_14_2008_8AM.trn'
GO

-- Two hours pass, another transaction log backup is made
BACKUP LOG TrainingDB
TO DISK = 'C:\Apress\Recipes\TrainingDB_Oct_14_2008_10AM.trn'
GO

USE master
GO
-- Kicking out all other connections
ALTER DATABASE TrainingDB
SET SINGLE_USER
WITH ROLLBACK IMMEDIATE

RESTORE DATABASE TrainingDB
FROM DISK = 'C:\Apress\Recipes\TrainingDB.bak'
WITH NORECOVERY, REPLACE

RESTORE LOG TrainingDB
FROM DISK = 'C:\Apress\Recipes\TrainingDB_Oct_14_2008_8AM.trn'
WITH NORECOVERY, REPLACE

RESTORE LOG TrainingDB
FROM DISK = 'C:\Apress\Recipes\TrainingDB_Oct_14_2008_10AM.trn'
WITH RECOVERY, REPLACE

BACKUP DATABASE TrainingDB
TO DISK = 'C:\Apress\Recipes\TrainingDB_Oct_14_2008.bak'

USE TrainingDB
GO
DELETE dbo.SalesOrderDetail
WHERE ProductID = 776
GO

SELECT GETDATE()
GO

BACKUP LOG TrainingDB
TO DISK = 'C:\Apress\Recipes\TrainingDB_Oct_14_2008_2022.trn'
USE master
GO
RESTORE DATABASE TrainingDB
FROM DISK = 'C:\Apress\Recipes\TrainingDB_Oct_14_2008.bak'
WITH FILE = 1, NORECOVERY,
STOPAT = '2008-10-14 20:18:56.583'
GO

RESTORE LOG TrainingDB
FROM DISK = 'C:\Apress\Recipes\TrainingDB_Oct_14_2008_2022.trn'
WITH RECOVERY,
STOPAT = '2008-10-14 20:18:56.583'
GO

USE TrainingDB
GO
SELECT COUNT(*)
FROM dbo.SalesOrderDetail
WHERE ProductID = 776
GO

-- Restoring a Database froma Differential Backup

USE master
GO
BACKUP DATABASE TrainingDB
TO DISK = 'C:\Apress\Recipes\TrainingDB_DiffExample.bak'

-- Time passes
BACKUP DATABASE TrainingDB
TO DISK = 'C:\Apress\Recipes\TrainingDB_DiffExample.diff'
WITH DIFFERENTIAL

-- More time passes
BACKUP LOG TrainingDB
TO DISK = 'C:\Apress\Recipes\TrainingDB_DiffExample_tlog.trn'

USE master
GO
-- Full database restore
RESTORE DATABASE TrainingDB
FROM DISK = 'C:\Apress\Recipes\TrainingDB_DiffExample.bak'
WITH NORECOVERY, REPLACE

-- Differential
RESTORE DATABASE TrainingDB
FROM DISK = 'C:\Apress\Recipes\TrainingDB_DiffExample.diff'
WITH NORECOVERY

-- Transaction log
RESTORE LOG TrainingDB
FROM DISK = 'C:\Apress\Recipes\TrainingDB_DiffExample_tlog.trn'
WITH RECOVERY

-- Restoring a File or Filegroup

USE master
GO
BACKUP DATABASE VLTestDB
FILEGROUP = 'FG2'
TO DISK = 'C:\Apress\Recipes\VLTestDB_FG2.bak'
WITH NAME = N'VLTestDB-Full Filegroup Backup',
SKIP, STATS = 20
GO

BACKUP LOG VLTestDB
TO DISK = 'C:\Apress\Recipes\VLTestDB_FG_Example.trn'

USE master
GO
RESTORE DATABASE VLTestDB
FILEGROUP = 'FG2'
FROM DISK = 'C:\Apress\Recipes\VLTestDB_FG2.bak'
WITH FILE = 1, NORECOVERY, REPLACE

RESTORE LOG VLTestDB
FROM DISK = 'C:\Apress\Recipes\VLTestDB_FG_Example.trn'
WITH FILE = 1, RECOVERY

-- Performing a Piecemeal (PARTIAL) Restore

USE master
GO
BACKUP DATABASE VLTestDB
FILEGROUP = 'PRIMARY'
TO DISK = 'C:\Apress\Recipes\VLTestDB_Primary_PieceExmp.bak'
GO

BACKUP DATABASE VLTestDB
FILEGROUP = 'FG2'
TO DISK = 'C:\Apress\Recipes\VLTestDB_FG2_PieceExmp.bak'
GO

BACKUP LOG VLTestDB
TO DISK = 'C:\Apress\Recipes\VLTestDB_PieceExmp.trn'
GO

RESTORE DATABASE VLTestDB
FILEGROUP = 'PRIMARY'
FROM DISK = 'C:\Apress\Recipes\VLTestDB_Primary_PieceExmp.bak'
WITH PARTIAL, NORECOVERY, REPLACE

RESTORE LOG VLTestDB
FROM DISK = 'C:\Apress\Recipes\VLTestDB_PieceExmp.trn'
WITH RECOVERY

USE VLTestDB
GO
SELECT name,
state_desc
FROM sys.database_files

-- Restoring a Page

BACKUP DATABASE TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_PageExample.bak'
GO

RESTORE DATABASE TestDB
PAGE='1:8'
FROM DISK = 'C:\Apress\Recipes\TestDB_PageExample.bak'
WITH NORECOVERY, REPLACE
GO

BACKUP LOG TestDB
TO DISK = 'C:\Apress\Recipes\TestDB_PageExample_tlog.trn'
GO

RESTORE LOG TestDB
FROM DISK = 'C:\Apress\Recipes\TestDB_PageExample_tlog.trn'
WITH RECOVERY

-- Identifying Databases withMultiple Recovery Paths

USE master
GO
IF NOT EXISTS (SELECT name
FROM sys.databases
WHERE name = 'RomanHistory')
BEGIN
CREATE DATABASE RomanHistory
END
GO

BACKUP DATABASE RomanHistory
TO DISK = 'C:\Apress\RomanHistory_A.bak'
GO

USE RomanHistory
GO
CREATE TABLE EmperorTitle
(EmperorTitleID int NOT NULL PRIMARY KEY IDENTITY(1,1),
TitleNM varchar(255))
GO

INSERT EmperorTitle (TitleNM)
VALUES ('Aulus'), ('Imperator'), ('Pius Felix'), ('Quintus')

BACKUP LOG RomanHistory
TO DISK = 'C:\Apress\RomanHistory_A.trn'
GO

SELECT last_log_backup_lsn LastLSN, recovery_fork_guid Rec_Fork,
first_recovery_fork_guid Frst_Fork, fork_point_lsn Fork_LSN
FROM sys.database_recovery_status
WHERE database_id = DB_ID('RomanHistory')

INSERT EmperorTitle (TitleNM)
VALUES ('Germanicus'), ('Lucius'), ('Maximus'), ('Titus')

BACKUP LOG RomanHistory
TO DISK = 'C:\Apress\RomanHistory_B.trn'
GO

USE master
GO
RESTORE DATABASE RomanHistory
FROM DISK = 'C:\Apress\RomanHistory_A.bak'
WITH NORECOVERY

RESTORE DATABASE RomanHistory
FROM DISK = 'C:\Apress\RomanHistory_A.trn'
WITH RECOVERY

SELECT last_log_backup_lsn LastLSN, recovery_fork_guid Rec_Fork,
first_recovery_fork_guid Frst_Fork, fork_point_lsn Fork_LSN
FROM sys.database_recovery_status
WHERE database_id = DB_ID('RomanHistory')


