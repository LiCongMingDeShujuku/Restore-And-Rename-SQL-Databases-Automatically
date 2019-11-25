![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 自动还原和重命名数据库
#### Restore And Rename SQL Databases Automatically
**发布-日期: 2014年05月30日 (评论)**

![#](images/##############?raw=true "#")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
此逻辑将自动使用新名称还原数据库。 需要做的就是提供所需的数据库名称和数据库备份文件的路径。逻辑中你可以看到还有一些额外的符号，这意味着需要取消注释行的位置以避免物理名称冲突。

如果要在还原逻辑运行之前用一些额外的逻辑来进行备份，请创建/添加类似这样的内容。

`declare @cmd varchar(max)
set @cmd = 'backup database ['' +  @database_name + ''] to disk = ''' + @backup_file_name  + ''' with format'
exec    (@cmd)`

这些是还原逻辑：


## English
This logic will automatically restore a database with a new name. All you need to do is provide the database name you want and the path for the database backup file. There’s some extra notations in the logic so you can see where you will need to uncomment a line in case to avoid physical name conflicts.

If you want to include some extra logic to make the backup before the restore logic runs, create/add something like this.

`declare @cmd varchar(max)
set @cmd = 'backup database ['' +  @database_name + ''] to disk = ''' + @backup_file_name  + ''' with format'
exec    (@cmd)`


Here’s the restore logic…

---
## Logic
```SQL
use master;
set nocount on
declare     @database_name          varchar (255)
declare     @backup_file_name       varchar (255)
declare     @create_restore_logic       varchar (max)
set     @database_name          = 'MyDatabaseName' -- Enter your database name here
set     @backup_file_name       = '\\MyNetworkShareLocation\MyDatabaseBackup.bak'  -- enter your backup file here
set     @create_restore_logic       = ''
select      @create_restore_logic       = @create_restore_logic + 
'
use master;
set nocount on
go
 
declare       @filelistonly        table
(
              logicalname          nvarchar     (128)
,             physicalname         nvarchar     (260)
,             [type]               char         (1)
,             filegroupname        nvarchar     (128)
,             size                 numeric      (20,0)
,             maxsize              numeric      (20,0)
,             fileid               bigint
,             createlsn            numeric      (25,0)
,             droplsn              numeric      (25,0)
,             uniqueid             uniqueidentifier
,             readonlylsn          numeric      (25,0)
,             readwritelsn         numeric      (25,0)
,             backupsizeinbytes    bigint
,             sourceblocksize      int
,             filegroupid          int
,             loggroupguid         uniqueidentifier
,             differentialbaselsn  numeric      (25,0)
,             differentialbaseguid uniqueidentifier
,             isreadonl            bit
,             ispresent            bit
,             tdethumbprint        varbinary    (32)
)
insert into 
              @filelistonly        exec         (''restore filelistonly from disk = ''''' + @backup_file_name + ''''''')
declare       @restore_line0       varchar      (255)
declare       @restore_line1       varchar      (255)
declare       @restore_line2       varchar      (255)
declare       @stats               varchar      (255)
declare       @move_files          varchar      (max)
set           @restore_line0       = (''use master; '')
set           @restore_line1       = (''exec master..sp_killallprocessindb ''''' + @database_name + ''''';'')
set           @restore_line2       = (''restore database [' + @database_name + '] from disk = ''''' + @backup_file_name + ''''' with replace, recovery, '')
set           @stats               = (''stats = 20;'')
set           @move_files          = ''''
select        @move_files          = @move_files + ''move '''''' + logicalname + '''''' to '''''' + physicalname + '''''','' + char(10) from @filelistonly order by fileid asc
--  if you are renaming the database you will need to comment out the line above, and uncomment the line below to avoid physicalname conflicts.

--如果要重命名数据库，则需要注释掉上面的行，并取消注释下面的行以避免物理名称冲突。
--  select        @move_files          = @move_files + ''move '''''' + logicalname + '''''' to '''''' + reverse(right(reverse(physicalname), len(physicalname) - charindex(''\'',reverse(physicalname),1) + 1)) + ' + @database_name + ' + ''_'' + fileid +  right(physicalname, 4)  + '''''','' + char(10) from @filelistonly order by fileid asc
exec
(
              @restore_line0
+             @restore_line1
+             @restore_line2
+             @move_files
+             @stats
)
go
'
 
-- uncomment the execution line below to run this logic.
--取消注释下面的执行行以运行此逻辑。

--  exec    (@create_restore_logic) 
select      (@create_restore_logic) for xml path (''), type


```
顺便说说，
刚发现一个错误，所以这是修复。
这将纠正运行此逻辑时可能出现的“名称”和“整数”错误。

By the way…
Just found a bug… So here’s the fix.
This will correct the ‘name’ & ‘integer’ error you might get when running this logic.

用这个新段替换旧段：
Replace the old segment with this new segment:

```SQL
--  if you are renaming the database you will need to comment out the line above, and uncomment the line below to avoid physical name conflicts.
--如果要重命名数据库，则需要注释掉上面的行，并取消注释下面的行以避免物理名称冲突。


-- select        @move_files          = @move_files + ''move '''''' + logicalname + '''''' to '''''' + reverse(right(reverse(physicalname), len(physicalname) - charindex(''\'',reverse(physicalname),1) + 1)) + 
''' + @database_name + ''' + ''_'' + cast(fileid as varchar(4)) +  right(physicalname, 4)  + '''''','' + char(10) from @filelistonly order by fileid asc

```

希望这对你有用。 (Hope this is useful.)


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

