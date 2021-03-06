
# **2_CleanScripts (Python):** Clean Up APS MPP Scripts Generated using  DWScripter 


## **How to Run the Program** ##


The program processing logic and information flow is illustrated in the diagram below: 

![Step 2: Clean Up APS MPP Scripts](/APS%20to%20SQL%20DW%20Migration%20-%20Schema%20and%20Data%20Migration%20with%20PolyBase/Images/2-CleanScripts.jpg)


Below are the steps to run the Python Program: 

**Step 1:** Create one configuration CSV file for the Python Program “CleanScripts.py”.  Refer the "Preparation Task: Configuration Driver CSV File Setup" after the steps for more details.  

**Step 2:** Run the CleanScripts.py and provide prompted info: The path and name of the configuration driver CSV file.

**Preparation Task: Configuration Driver CSV File Setup**

Create the configuration driver CSV file by referring the definitions below. Sample CSV configuration files are provided to aid this preparation task. 


| Parameter           | Purpose                              |      Value (Sample)     |
| --------------------| -------------------------------------|-------------------------| 
| Active              | 1 – Run line, 0 – Skip line.         | 0 or 1                  |
| SicDir              | Directory where the APS Scripts resides. This should be the output file directories from Step 1: CreateMPPScripts. Must have “\” on end. | C:\APS2SQLDW\Output\1_CreateMPPScripts\adventure_works\Tables\ |
| OutDir        | Output director of this step, where the cleaned scripts will reside. Must have “\” on end. | C:\APS2SQLDW\Output\2_CleanScripts\adventure_works\Tables\        |
| ObjectType        | Type of the Scripts      | TABLE, VIEW, SP  (case insensitive)   |


## **What the Program(s) Does** ##

When the DWScripter program creates the table scripts, additional details are added to the script that are unnecessary for every table, view, or stored procedure created and need to be removed.  The Stats and Index creation statements should be removed from the Create Table DDL and placed in separate script files to be applied after the data has been loaded.  Creating the index/stats and then loading the data into SQLDW will slow down the data loading and the table creation.  

This Python Program (CleanScripts.py) removes unnecessary lines generated by the DWScripter program. It also separates the Index Creation Statement(s) and Statistics Creation Statement(s) from the Table Creation DDL Statement, then stores "Create Index" and "Create Statistics" statement(s) in separate ".dsql" file with the prefix "IDXS_" or "STATS_" respectively. For example, a separate Index Creation file will be named as "IDXS_SchemaName_TableName_DDL.dsql", the "Create Table" Statement will be stored as "SchemaName_TableName_DDL.dsql".

The table below describes what is changed by this program and why. 

| Text                | Changes by the Clean Up Program      |      Reason for the Changes                             |
| --------------------| -------------------------------------|---------------------------------------------------------| 
| --Header Comments   |   Deleted                            | Not needed                                              |
| Use DBName          |   Deleted                            | SQLDW does not support USE                              |
| Go                  |   Deleted                            | Not needed                                              |
| Create Schema       |   Deleted                            | These do not need to be created in every table.         |
| Create Index        |   Move to separate script            | Create Index statements should be run after the initial load of data has occurred |
| Create Statistics   |   Move to separate script     | Create Statistics statements should be run after the initial load of data has occurred     |
| ;                   | Deleted | Delete all ; as this will cause the deployment script to fail as it is expecting only a single statement to be run. |

Using an example T-SQL DDL script, the 'before' and 'after' DDLs are illustrated below. 
Example T-SQL Create Table Statement DDL file "IDXS_testSchema_TblIndexTwo_DDL.dsql" before "cleaning up":

    -- script generated the : 11/7/2018 18:59:49
    -- objects scripted - tables = 1 - schemas = 1
    USE adventure_works
    GO
    
    CREATE SCHEMA [testSchema];
    GO
    
    CREATE TABLE [testSchema].[TblStatsTwoIndex]
    (
    	[Source]	varchar	(3)	COLLATE	Latin1_General_100_CI_AS_KS_WS	NOT NULL 
    	,[CustomerKey]	nvarchar	(12)	COLLATE	Latin1_General_100_CI_AS_KS_WS	NULL 
    	,[License_Number]	nvarchar	(15)	COLLATE	Latin1_General_100_CI_AS_KS_WS	NULL 
    	,[Customer_Number]	nvarchar	(10)	COLLATE	Latin1_General_100_CI_AS_KS_WS	NULL 
    )
    WITH ( DISTRIBUTION = HASH ([CustomerKey])
    , CLUSTERED INDEX ([License_Number]
    ,[Customer_Number]));
    CREATE INDEX [Index_TblStatsTwoIndex_CustomerKey] ON [adventure_works].[testSchema].[TblStatsTwoIndex] 
    ([CustomerKey]);
    CREATE STATISTICS [Stats_TblStatsTwoIndex_CustomerKey] ON [testSchema].[TblStatsTwoIndex] 
    ([CustomerKey] )
    ;
    CREATE STATISTICS [Stats_TblStatsTwoIndex_CustomerNum] ON [testSchema].[TblStatsTwoIndex] 
    ([Customer_Number] )
    ;

After the being processed by the "Cleanup" program, the above T-SQL DDL file is "cleaned up" and also separated into three different DDL files: 

(1) Create Table Statement is now stored into a new file into a new folder with the same file name: "testSchema_TblIndexTwo_DDL.dsql":
    
    CREATE TABLE [testSchema].[TblStatsTwoIndex]
    (
    	[Source]	varchar	(3)	COLLATE	Latin1_General_100_CI_AS_KS_WS	NOT NULL 
    	,[CustomerKey]	nvarchar	(12)	COLLATE	Latin1_General_100_CI_AS_KS_WS	NULL 
    	,[License_Number]	nvarchar	(15)	COLLATE	Latin1_General_100_CI_AS_KS_WS	NULL 
    	,[Customer_Number]	nvarchar	(10)	COLLATE	Latin1_General_100_CI_AS_KS_WS	NULL 
    )
    WITH ( DISTRIBUTION = HASH ([CustomerKey])
    , CLUSTERED INDEX ([License_Number]
    ,[Customer_Number]))
    

(2) Create Index Statement is now stored into a Separate File "IDXS_testSchema_TblIndexTwo_DDL.dsql"

    CREATE INDEX [Index_TblIndexTwo_CustomerKey] ON [adventure_works].[testSchema].[TblIndexTwo] 
    ([CustomerKey])
    CREATE INDEX [Index_TblIndexTwo_CustomerNum] ON [adventure_works].[testSchema].[TblIndexTwo] 
    ([Customer_Number])

(3) Create Statistics Statement is now stored into a Separate File "STATS_testSchema_TblStatsTwoIndex_DDL.dsql"

    CREATE STATISTICS [Stats_TblStatsTwoIndex_CustomerKey] ON [testSchema].[TblStatsTwoIndex] 
    ([CustomerKey] )
    
    CREATE STATISTICS [Stats_TblStatsTwoIndex_CustomerNum] ON [testSchema].[TblStatsTwoIndex] 
    ([Customer_Number] )
    
