# P20X Data + Analytics Session Hands-on Lab

<br>
##Lab Objectives

1.  Create SQL Data Warehouse using Azure Portal and connect with SQL Server Management Studio.

2.  Setup the connectivity between centralized Azure blob store with Azure SQL Data Warehouse using Polybase to load non-relational blob data into relational Data Warehouse.

3.  Load data from Azure SQL Data Warehouse into Power BI Desktop to create interactive charts and visualizations.

##Software Prerequisites

-   Windows 10 PC

-   [Free MS FTE MSDN Subscription](http://subscriptions/)

-   [SQL Server Management Studio](http://go.microsoft.com/fwlink/?LinkID=822301)

-   [Power BI Desktop](https://go.microsoft.com/fwlink/?LinkId=521662&clcid=0x409)

##Create SQL Data Warehouse using Azure Portal

1.  Sign in to your Azure Subscription.

2.  Click **+ New** &gt; **Data + Storage** &gt; **SQL Data Warehouse**.

	 <img src="./media/image1.png" width="624" height="400" />

3.  In the **SQL Data Warehouse** blade, fill in the following information:

	-   **Server**: It is recommended to select your SQL server first. You can select an existing server or [create a new one](https://azure.microsoft.com/en-us/documentation/articles/sql-data-warehouse-get-started-new-server/).

	> <img src="./media/image2.PNG" width="647" height="634" />

	-   **Database name**: The name that will be used to reference the SQL Data Warehouse. It must be unique to the server.

	-   **Performance**: It is recommended to start with 400 DWUs. You can move the slider to the left or right to adjust the performance of your data warehouse, or scale up or down after creation. To learn more about DWUs see the documentation on [scaling](https://azure.microsoft.com/en-us/documentation/articles/sql-data-warehouse-manage-compute-overview/) or our [pricing page](https://azure.microsoft.com/pricing/details/sql-data-warehouse/).

	-   **Subscription**: Select the subscription that this SQL Data Warehouse will bill to.

	-   **Resource group**: Resource groups are containers designed to help you manage a collection of Azure resources. The best practice is to place all the resources in a single resource group and since we are in Colorado, create all the resources in West US location

	-   **Select source**: Click **Select source** &gt; **Blank database**.

	<img src="./media/image3.PNG" width="619" height="665" />
<br>

4.  Click **Create** to create your SQL Data Warehouse.

5.  Wait for a few minutes and your SQL Data Warehouse will be ready. When finished, you should be returned to the portal. You can find your SQL Data Warehouse on your dashboard, or listed under your SQL Databases, or in the resource group that you used to create it.

	<img src="./media/image4.png" width="624" height="399" />
<br>

6.  Use the following steps in the Azure portal to create a server-level firewall rule that allows connections from an individual IP address (your client computer).

    1.  Click the SQL server we just created to create a firewall rule.

    2.  Click **show firewall settings**

	<img src="./media/image5.png" width="624" height="426" />
<br>

7.  Click **Add Client IP** to have Azure create a rule for your client's IP address.

	<img src="./media/image6.png" width="624" height="393" />
<br>

8.  Click **Save** to create the server-level firewall rule.

	<img src="./media/image7.PNG" width="590" height="375" />

Now that you have created a SQL Data Warehouse, you are ready to connect and begin querying.

<br>

##Setup the PolyBase connectivity to centralized Azure Blob Store

PolyBase is a technology that accesses and combines both non-relational and relational data, all from within SQL Server. It allows you to run queries on external data in Hadoop or Azure blob storage. By simply using Transact-SQL (T-SQL) statements, you an import and export data back and forth between relational tables in SQL Server and non-relational data stored in Hadoop or Azure Blob Storage. You can also query the external data from within a T-SQL query and join it with relational data.

<br>
<img src="./media/image8.png" width="624" height="276" />
<br>

###Accessing the centralized blob storage:
---------------------------------------

The centralized data store for our workshop is blob storage. To access this store, open **Azure Storage Explorer** and add a storage account. Following are the **access details:**

**Storage account name:** p20xdatastorage

**Storage account name:** 2NX7xVaHL2ZQTPc0tBdIOxpz18zyMudmAlLNxECjFSE01M5551XcWgRIPzpiTHhEvHeTUEj8Z72Oh4XpqAdoYA==

<br>
<img src="./media/image9.PNG" width="624" height="404" />

Explore the 3 blob storage containers.

-   **Rawdata**: This is the raw data access

-   **Load**: This is the container structure to load all the files using PolyBase

-   **POCFiles**: This container includes a ZIP file with all the scripts/dashboards etc.

<img src="./media/image10.PNG" width="409" height="328" />

We will use PolyBase to load this data into SQL Data Warehouse database.

###Accessing Azure SQL DW from SQL Server Management Studio:

Your SQL Data Warehouse is associated with an Azure SQL server. To connect you need the fully qualified name of the server (**servername**.database.windows.net).

Click **SQL databases** and click the database you want to connect to. Locate the full server name and copy that name to your clipboard.

<img src="./media/image11.png" width="624" height="364" />

To start querying your data warehouse, **open SQL Server Management Studio** and **connect to server. **

-   **Server name** - Enter the **server name** previously identified.

-   **Authentication** - Select **SQL Server Authentication** 

-   **User Name** and **Password -** Enter user name and password that you typed while creating the server during SQL data warehouse creation process.

<img src="./media/image12.PNG" width="542" height="321" />

To explore, expand your Azure SQL server. You can view the databases associated with the server.

<br>
<img src="./media/image13.PNG" width="399" height="386" />
<br>

###Create an external table for the blob storage data:

In this section we create external tables that defines our data.

PolyBase uses external tables to access data in Azure blob storage. Since the data is not stored within SQL Data Warehouse, PolyBase handles authentication to the external data by using a database-scoped credential.

Select **New Query** in SQL Server Management Studio. **Change the database name on the top left from ‘master’ to your database** and copy the following SQL queries in the query window:

***NOTE:*** For query reference -

-   [Create Master Key (T-SQL)](https://msdn.microsoft.com/library/ms174382.aspx) to encrypt the secret of your database scoped credential.

-   [Create Database Scoped Credential (T-SQL)](https://msdn.microsoft.com/library/mt270260.aspx) to specify authentication information for your Azure storage account.

-   [Create External Data Source (T-SQL)](https://msdn.microsoft.com/library/dn935022.aspx) to specify the location of your Azure blob storage.

-   [Create External File Format (T-SQL)](https://msdn.microsoft.com/library/dn935026.aspx) to specify the format of your data.

-   [Create External Table (T-SQL)](https://msdn.microsoft.com/library/dn935021.aspx) to specify the table definition and location of the data.

-- 1) Create a master key.

-- Only necessary if one does not already exist.

-- Required to encrypt the credential secret in the next step.

CREATE MASTER KEY;

GO

-- 2) Create a database scoped credential

-- IDENTITY: Provide any string, it is not used for authentication to Azure storage.

-- SECRET: Storage API Key to the Centralized Storage Account

CREATE DATABASE SCOPED CREDENTIAL AzureStorageCredentialsInstructurePoc

WITH

IDENTITY = 'user',

SECRET = '2NX7xVaHL2ZQTPc0tBdIOxpz18zyMudmAlLNxECjFSE01M5551XcWgRIPzpiTHhEvHeTUEj8Z72Oh4XpqAdoYA==';

GO

SELECT \* FROM SYS.DATABASE\_CREDENTIALS;

GO

-- 3) Create an External Data Source

-- LOCATION: WASB (Blob) location

-- CREDENTIAL: Auth created above

CREATE EXTERNAL DATA SOURCE AzureStorageInstructurePoc

WITH (

TYPE = Hadoop,

LOCATION = 'wasbs://load@p20xdatastorage.blob.core.windows.net',

CREDENTIAL = AzureStorageCredentialsInstructurePoc

);

GO

SELECT \*FROM SYS.EXTERNAL\_DATA\_SOURCES;

GO

-- 4) Create a comma delimted file format

-- FORMAT\_OPTIONS: comma delimated with string delimiters

CREATE EXTERNAL FILE FORMAT TextFileTab

WITH (

FORMAT\_TYPE = DelimitedText,

FORMAT\_OPTIONS (FIELD\_TERMINATOR = '\\t')

);

GO

SELECT \* FROM SYS.EXTERNAL\_FILE\_FORMATS;

GO

Canvas Network (http://canvas.net) is an open online course website operated by the
educational technology company Instructure. Canvas Network hosts open online courses,
including Massive Open Online Courses (MOOCs), that are freely available to participants
around the world. Data about the open online courses and users' experiences in those courses
may be beneficial to researchers seeking to better understand and describe the MOOC
phenomenon, as well as to gain insight into online learning behaviors and course design
characteristics.

-- 5) Create external tables for courses and enrollments

if (object\_id('dbo.CourseDimExternal') is NOT NULL)

begin

drop external table dbo.CourseDimExternal;

end

go

CREATE EXTERNAL TABLE dbo.CourseDimExternal (

Id varchar(100) NOT NULL,

Name varchar(100) NULL,

Type varchar(100) NULL,

CreatedAt varchar(100) NULL,

StartAt varchar(100) NULL,

ConcludeAt varchar(100) NULL,

Wiki\_Id varchar(100) NULL

)

WITH (

LOCATION='coursedim/',

DATA\_SOURCE=AzureStorageInstructurePoc,

FILE\_FORMAT=TextFileTab,

REJECT\_TYPE = Value,

REJECT\_VALUE = 100000

);

Go

SELECT \* FROM dbo.CourseDimExternal;

Go

if (object\_id('dbo.EnrollmentDimExternal') is NOT NULL)

begin

drop external table dbo.EnrollmentDimExternal;

end

go

CREATE EXTERNAL TABLE dbo.EnrollmentDimExternal (

Id varchar(100) NOT NULL,

Type varchar(100) NULL,

Workflow\_State varchar(100) NULL,

CreatedAt varchar(100) NULL,

UpdatedAt varchar(100) NULL,

StartAt varchar(100) NULL,

EndAt varchar(100) NULL,

ConcludeAt varchar(100) NULL,

Course\_Id varchar(100) NULL,

User\_Id varchar(100) NULL

)

WITH (

LOCATION='enrollmentdim/',

DATA\_SOURCE=AzureStorageInstructurePoc,

FILE\_FORMAT=TextFileTab

);

go

SELECT \* FROM dbo.EnrollmentDimExternal;

go

if (object\_id('dbo.EnrollmentFactExternal') is NOT NULL)

begin

drop external table dbo.EnrollmentFactExternal;

end

go

CREATE EXTERNAL TABLE dbo.EnrollmentFactExternal (

Enrollment\_Id varchar(100) NOT NULL,

User\_Id varchar(100) NULL,

Course\_Id varchar(100) NULL,

Computed\_Final\_Score varchar(100) NULL

)

WITH (

LOCATION='enrollmentfact/',

DATA\_SOURCE=AzureStorageInstructurePoc,

FILE\_FORMAT=TextFileTab

);

Go

SELECT count(\*) FROM dbo.EnrollmentFactExternal;

go

Load data into SQL Data warehouse

Once the external table is created, you can either load the data into a new table in the data warehouse or insert it into an existing table.

-   To load the data into a new table, run the [CREATE TABLE AS SELECT (T-SQL)](https://msdn.microsoft.com/library/mt204041.aspx)statement. The new table will have the columns named in the query. The data types of the columns will match the data types in the external table definition.

-   To load the data into an existing table, use the [INSERT...SELECT (T-SQL)](https://msdn.microsoft.com/library/ms174335.aspx) statement.

-- 6) Create internal SQL Server tables for Course and Enrollment to load data

-- Note the hash distribution

if (object\_id('dbo.CourseMaster') is NOT NULL)

begin

drop table dbo.CourseMaster;

end

go

CREATE TABLE dbo.CourseMaster

(

Course\_Id varchar(100) NOT NULL,

Name varchar(100) NULL,

Type varchar(100) NULL,

Created\_At datetime NOT NULL,

StartAt varchar(100) NULL,

ConcludeAt varchar(100) NULL,

Wiki\_Id varchar(100) NULL

)

WITH

(

DISTRIBUTION = HASH (Course\_Id),

CLUSTERED COLUMNSTORE INDEX

);

go

-- 7) Truncate the internal table and load all the records from external table

-- Since the table is already created, use “insert t-sql” to load the data

truncate table dbo.CourseMaster;

go

set nocount on;

insert into dbo.CourseMaster

SELECT distinct

Id, Name, Type,

convert(smalldatetime,

substring(CreatedAt, 1, 4) + '-' + substring(CreatedAt, 6, 2) + '-' + substring(CreatedAt, 9, 2)

) as Created\_At, StartAt, ConcludeAt, Wiki\_Id

FROM

dbo.CourseDimExternal

where Id != 'Id'

AND isDate(substring(CreatedAt, 1, 4) + '-' + substring(CreatedAt, 6, 2) + '-' + substring(CreatedAt, 9, 2)) = 1

SELECT \* FROM dbo.CourseMaster

---- Follow the same procedure for Enrollments too

if (object\_id('dbo.EnrollmentMaster') is NOT NULL)

begin

drop table dbo.EnrollmentMaster;

end

go

CREATE TABLE dbo.EnrollmentMaster

(

Enrollment\_Id varchar(100) NOT NULL,

Type varchar(50) NULL,

User\_Id varchar(50) NULL,

Course\_Id varchar(50) NULL,

Created\_At datetime NULL,

Computed\_Final\_Score decimal(12,4) NULL

)

WITH

(

DISTRIBUTION = HASH (Enrollment\_Id),

CLUSTERED COLUMNSTORE INDEX

);

go

truncate table dbo.EnrollmentMaster;

go

set nocount on;

insert into dbo.EnrollmentMaster

SELECT distinct

Id, ed.Type, ed.User\_Id, --ef.User\_id,

ed.Course\_Id, --ef.Course\_id,

convert(smalldatetime,

substring(CreatedAt, 1, 4) + '-' + substring(CreatedAt, 6, 2) + '-' + substring(CreatedAt, 9, 2)

) as Created\_At,

case when (ef.Computed\_Final\_Score = '\\N') then NULL else ef.Computed\_Final\_Score end

FROM

dbo.EnrollmentDimExternal ed inner join dbo.EnrollmentFactExternal ef on (ed.Id = ef.Enrollment\_Id)

where Id != 'Id'

AND isDate(substring(CreatedAt, 1, 4) + '-' + substring(CreatedAt, 6, 2) + '-' + substring(CreatedAt, 9, 2)) = 1

SELECT \* FROM dbo.EnrollmentMaster;

Now, let’s do some data analysis by writing some queries with the tables we have. The following queries give us insight into the enrollments and student grades.

--check the type of enrollments

select distinct(type) from \[dbo\].\[EnrollmentMaster\]

--Student count

select count(\*) as StudentCount from \[dbo\].\[EnrollmentMaster\] where type = 'StudentEnrollment'

--Teacher count

select count(\*) as TeacherCount from \[dbo\].\[EnrollmentMaster\] where type = 'TeacherEnrollment'

--Count of Students with grades under 70

select count(\*) as StudentGradesUnder70 from \[dbo\].\[EnrollmentMaster\]

where type = 'StudentEnrollment' and Computed\_Final\_Score &lt; 0.7 and Computed\_Final\_Score is not null

--Percentage of Students with grades under 70

select (100\*count(\*)/(select count(\*) from \[dbo\].\[EnrollmentMaster\] where type = 'StudentEnrollment' and Computed\_Final\_Score is not null)) as Percentage

from \[dbo\].\[EnrollmentMaster\]

where type = 'StudentEnrollment' and Computed\_Final\_Score &lt; 0.7 and Computed\_Final\_Score is not null

##Loading the data into Power BI Desktop for Visualization

Power BI integration allows you to directly connect to the data stored in your Azure SQL Data Warehouse offering data exploration, dynamic reporting and visualization.

1.  Open Power BI Desktop. It allows you to easily connect to the ever expanding world of data.

2.  There are *all sorts* of data sources available in Power BI Desktop. The following image shows how to connect to data, by selecting the **File** ribbon, then **Get Data &gt; Azure**.

    select **'Azure SQL Data Warehouse'** and then click **'Connect'**.

	<img src="./media/image14.PNG" width="624" height="577" />
	<br>
3.  Enter the necessary connection information. Your server name and database name can be found in the Azure Portal.

	<img src="./media/image15.PNG" width="624" height="315" />

4.  When you select **OK**, the **Navigator** window returns the tables from your Azure SQL data warehouse. In this case, it returns and lists 5 tables. Select the internal tables – **Course Master** and **Enrollment Master**. The **Navigator** window displays a preview.

    <img src="./media/image16.PNG" width="624" height="491" />
	<br>
5.  At this point we can edit the query before loading the table, by selecting **Edit** from the bottom of the window, or we can load the table. Click **‘Load’** to load the table.

6.  Now, we are on the **Report View** where you create reports and visualizations by dragging values into fields. You can also find **Data View** and **Relationship View** on the left pane. Explore them to get an initial idea of your data.

    <img src="./media/image17.PNG" width="49" height="141" />

 	**Data View** helps you inspect, explore, and understand data in your Power BI Desktop model. It's different from how you view tables, columns, and data in **Query Editor**. With Data View, you’re looking at your data *after* it has been loaded into the model. You can also make sequence of transformations to the data.

 	<img src="./media/image18.png" width="624" height="257" />

 	**Relationship View** shows all of the tables, columns, and relationships in your model. This can be especially helpful when your model has complex relationships between many tables.

 	<img src="./media/image19.PNG" width="624" height="477" />

	<br>

7.  Now, on the Report view, let’s start off with creating some simple visualizations. To find the count of the most important thing you want to track, **Card chart** and **Gauge chart** are useful.

    **NOTE**: Sometimes a single number is the most important thing you want to track in your Power BI dashboard, such as total sales, market share year over year, or total opportunities. You can create a big number tile in Power BI using a **Card chart**. A **Gauge chart** has a circular arc and displays a single value that measures progress toward a goal/KPI. 

	-   Create a Card chart from the **Visualizations** pane on the report and drag Course\_Id of the CourseMaster from the **Fields** pane.

	-   Similarly, create a Gauge chart from the **Visualizations** pane on the report and drag Enrollment\_Id of the EnrollmentMaster from the **Fields** pane.

	-   Here, we are tracking the count of Courses and the progress with Enrollments

    <img src="./media/image20.PNG" width="624" height="565" />
	<br>

8. You can also format and customize your chart by selecting the **‘format’** brush icon the visualization pane. For Card and Gauge charts, you can add a title, change colors, increase font-size of the text and so on.
	
	<br>
	<img src="./media/image21.PNG" width="624" height="647" />
	<br>

9. Now, let’s add a **Line chart** to **analyze the number of students enrolled over time**. Line charts are used to emphasize the overall shape of an entire series of values, usually over time.

	-   Create a Line chart from the **Visualizations** pane on the report.

	-   From the EnrollmentMaster table, drag

    -   Created\_At column to Axis,

    -   Type column to Legend

    -   User\_Id column to Values

	-   Since we are only interested in students’ enrollment, we can filter that by expanding Type column in the **‘Filters’** section and selecting only StudentEnrollment Value.

	-   Now, repeat this process and add another Line chart to analyze the number of teachers enrolled over time.

	<br>
	<img src="./media/image22.png" width="624" height="505" />
	<br>

10. We can add a slicer to narrow the portion of the dataset shown in the other visualizations on the report page.  Slicers are an alternate way of filtering.

	-   From the Fields pane, select **CourseMaster &gt;** **Name.**

	-   Convert the visualization to a slicer. In the Visualizations pane, select the slicer icon.

	<br>
	<img src="./media/image23.png" width="624" height="289" />
	<br> 

	-   Now, slice the report page for **Business and Management** course. **Notice how other visualizations get updated to reflect these selections.** Also, select other courses to explore and analyze more course-specific information.

	<br>
	<img src="./media/image24.png" width="624" height="303" />
	<br>

11. Finally, we have a report. You can **rename** your report at the bottom left corner of the page. Also, you can **publish** your reports to **Power BI Service** by clicking ‘**Publish’** under Home menu. You can always edit and republish your reports to update the changes.


	<br>
	<img src="./media/image25.png" width="624" height="516" />



