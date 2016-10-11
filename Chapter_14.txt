-----------------------------------------------------------------------
-- Source Code: SQL Server 2008 Transact-SQL Recipes, Joseph Sack
-----------------------------------------------------------------------
-- Do not execute the following code in a single batch.  These samples
-- are provided in order to follow along with specific recipes.  
-----------------------------------------------------------------------

-- Creating XML Data Type Columns

IF NOT EXISTS (SELECT name FROM sys.databases
WHERE name = 'TestDB')
BEGIN
CREATE DATABASE TestDB
END
GO

USE TestDB
GO
CREATE TABLE dbo.Book
(BookID int IDENTITY(1,1) PRIMARY KEY,
ISBNNBR char(10) NOT NULL,
BookNM varchar(250) NOT NULL,
AuthorID int NOT NULL,
ChapterDESC XML NULL)
GO

DECLARE @Book XML
SET @Book =
CAST('<Book name="SQL Server 2000 Fast Answers">
<Chapters>
<Chapter id="1"> Installation, Upgrades... </Chapter>
<Chapter id="2"> Configuring SQL Server </Chapter>
<Chapter id="3"> Creating and Configuring Databases </Chapter>
<Chapter id="4"> SQL Server Agent and SQL Logs </Chapter>
</Chapters>
</Book>' as XML)

CREATE PROCEDURE dbo.usp_INS_Book
@ISBNNBR char(10),
@BookNM varchar(250),
@AuthorID int,
@ChapterDESC xml
AS
INSERT dbo.Book
(ISBNNBR, BookNM, AuthorID, ChapterDESC)
VALUES (@ISBNNBR, @BookNM, @AuthorID, @ChapterDESC)
GO

-- Inserting XML Data into a Column

INSERT dbo.Book
(ISBNNBR, BookNM, AuthorID, ChapterDESC)
VALUES ('570X000000',
'SQL Server 2008 T-SQL Recipes',
55,
CAST('<Book name="SQL Server 2008 T-SQL Recipes">
<Chapters>
<Chapter id="1"> SELECT </Chapter>
<Chapter id="2"> INSERT,UPDATE,DELETE </Chapter>
<Chapter id="3"> Transactions, Locking, Blocking, and Deadlocking </Chapter>
<Chapter id="4"> Tables </Chapter>
<Chapter id="5"> Indexes </Chapter>
<Chapter id="6"> Full-text search </Chapter>
</Chapters>
</Book>' as XML))

DECLARE @Book XML
SET @Book =
CAST('<Book name="SQL Server 2000 Fast Answers">
<Chapters>
<Chapter id="1"> Installation, Upgrades... </Chapter>
<Chapter id="2"> Configuring SQL Server </Chapter>
<Chapter id="3"> Creating and Configuring Databases </Chapter>
<Chapter id="4"> SQL Server Agent and SQL Logs </Chapter>
</Chapters>
</Book>' as XML)

INSERT dbo.Book
(ISBNNBR, BookNM, AuthorID, ChapterDESC)
VALUES ('1590591615',
'SQL Server 2000 Fast Answers',
55,
@Book)

-- Validating XML Data Using Schemas

CREATE XML SCHEMA COLLECTION BookStoreCollection
AS
N'<xsd:schema targetNamespace="http://JOEPROD/BookStore"
xmlns:xsd="http://www.w3.org/2001/XMLSchema"
xmlns:sqltypes="http://schemas.microsoft.com/sqlserver/2004/sqltypes"
elementFormDefault="qualified">
<xsd:import namespace=
"http://schemas.microsoft.com/sqlserver/2004/sqltypes" />
<xsd:element name="Book">
<xsd:complexType>
<xsd:sequence>
<xsd:element name="BookName" minOccurs="0">
<xsd:simpleType>
<xsd:restriction base="sqltypes:varchar">
<xsd:maxLength value="50" />
</xsd:restriction>
</xsd:simpleType>
</xsd:element>
<xsd:element name="ChapterID" type="sqltypes:int"
minOccurs="0" />
<xsd:element name="ChapterNM" minOccurs="0">
<xsd:simpleType>
<xsd:restriction base="sqltypes:varchar">
<xsd:maxLength value="50" />
</xsd:restriction>
</xsd:simpleType>
</xsd:element>
</xsd:sequence>
</xsd:complexType>
</xsd:element>
</xsd:schema>'

SELECT name
FROM sys.XML_schema_collections
ORDER BY create_date

SELECT n.name
FROM sys.XML_schema_namespaces n
INNER JOIN sys.XML_schema_collections c ON
c.XML_collection_id = n.XML_collection_id
WHERE c.name = 'BookStoreCollection'

CREATE TABLE dbo.BookInfoExport
(BookID int IDENTITY(1,1) PRIMARY KEY,
ISBNNBR char(10) NOT NULL,
BookNM varchar(250) NOT NULL,
AuthorID int NOT NULL,
ChapterDESC xml (BookStoreCollection) NULL)

DECLARE @Book XML (DOCUMENT BookStoreCollection)

-- Retrieving XML Data

CREATE TABLE dbo.BookInvoice
(BookInvoiceID int IDENTITY(1,1) PRIMARY KEY,
BookInvoiceXML XML NOT NULL)
GO

INSERT dbo.BookInvoice
(BookInvoiceXML)
VALUES ('<BookInvoice invoicenumber="1" customerid="22" orderdate="7/1/2008">
<OrderItems>
<Item id="22" qty="1" name="SQL Fun in the Sun"/>
<Item id="24" qty="1" name="T-SQL Crossword Puzzles"/>
</OrderItems>
</BookInvoice>')

INSERT dbo.BookInvoice
(BookInvoiceXML)
VALUES ('<BookInvoice invoicenumber="1" customerid="40" orderdate="7/11/2008">
<OrderItems>
<Item id="11" qty="1" name="MCDBA Cliff Notes"/>
</OrderItems>
</BookInvoice>')

INSERT dbo.BookInvoice
(BookInvoiceXML)
VALUES ('<BookInvoice invoicenumber="1" customerid="9" orderdate="7/22/2008">
<OrderItems>
<Item id="11" qty="1" name="MCDBA Cliff Notes"/>
<Item id="24" qty="1" name="T-SQL Crossword Puzzles"/>
</OrderItems>
</BookInvoice>')

SELECT BookInvoiceID
FROM dbo.BookInvoice
WHERE BookInvoiceXML.exist
('/BookInvoice/OrderItems/Item[@id=11]') = 1

DECLARE @BookInvoiceXML xml

SELECT @BookInvoiceXML = BookInvoiceXML
FROM dbo.BookInvoice
WHERE BookInvoiceID = 2

SELECT BookID.value('@id','integer') BookID
FROM @BookInvoiceXML.nodes('/BookInvoice/OrderItems/Item')
AS BookTable(BookID)

DECLARE @BookInvoiceXML XML

SELECT @BookInvoiceXML = BookInvoiceXML
FROM dbo.BookInvoice
WHERE BookInvoiceID = 3

SELECT @BookInvoiceXML.query('/BookInvoice/OrderItems')

SELECT DISTINCT
BookInvoiceXML.value
('(/BookInvoice/OrderItems/Item/@name)[1]', 'varchar(30)') as BookTitles
FROM dbo.BookInvoice
UNION
SELECT DISTINCT
BookInvoiceXML.value
('(/BookInvoice/OrderItems/Item/@name)[2]', 'varchar(30)')
FROM dbo.BookInvoice

-- Modifying XML Data

UPDATE dbo.BookInvoice
SET BookInvoiceXML.modify
('insert <Item id="920" qty="1" name="SQL Server 2008 Transact-SQL Recipes"/>
into (/BookInvoice/OrderItems)[1]')
WHERE BookInvoiceID = 2

SELECT BookInvoiceXML
FROM dbo.BookInvoice
WHERE BookInvoiceID = 2

-- Indexing XML Data

CREATE PRIMARY XML INDEX idx_XML_Primary_Book_ChapterDESC
ON dbo.Book(ChapterDESC)
GO

CREATE XML INDEX idx_XML_Value_Book_ChapterDESC
ON dbo.Book(ChapterDESC)
USING XML INDEX idx_XML_Primary_Book_ChapterDESC
FOR VALUE
GO

SELECT name, secondary_type_desc
FROM sys.XML_indexes
WHERE object_id = OBJECT_ID('dbo.Book')

-- Formatting Relational Data As XML

USE AdventureWorks
GO
SELECT ShiftID, Name
FROM HumanResources.Shift
FOR XML RAW('Shift'), ROOT('Shifts'), TYPE

SELECT TOP 3 BusinessEntityID,
Shift.Name,
Department.Name
FROM HumanResources.EmployeeDepartmentHistory Employee
INNER JOIN HumanResources.Shift Shift ON
Employee.ShiftID = Shift.ShiftID
INNER JOIN HumanResources.Department Department ON
Employee.DepartmentID = Department.DepartmentID
ORDER BY BusinessEntityID
FOR XML AUTO, TYPE

SELECT TOP 3
Shift.Name,
Department.Name,
BusinessEntityID
FROM HumanResources.EmployeeDepartmentHistory Employee
INNER JOIN HumanResources.Shift Shift ON
Employee.ShiftID = Shift.ShiftID
INNER JOIN HumanResources.Department Department ON
Employee.DepartmentID = Department.DepartmentID
ORDER BY Shift.Name, Department.Name, BusinessEntityID
FOR XML AUTO, TYPE

SELECT TOP 3
1 AS Tag,
NULL AS Parent,
BusinessEntityID AS [Vendor!1!VendorID],
Name AS [Vendor!1!VendorName!ELEMENT],
CreditRating AS [Vendor!1!CreditRating]
FROM Purchasing.Vendor
ORDER BY CreditRating
FOR XML EXPLICIT, TYPE

SELECT Name as "@Territory",
CountryRegionCode as "@Region",
SalesYTD
FROM Sales.SalesTerritory
WHERE SalesYTD > 6000000
ORDER BY SalesYTD DESC
FOR XML PATH('TerritorySales'), ROOT('CompanySales'), TYPE

-- Converting XML to a Relational Form

CREATE PROCEDURE dbo.usp_SEL_BookXML_Convert_To_Relational
@XMLDoc xml
AS
DECLARE @docpointer int
EXEC sp_XML_preparedocument @docpointer OUTPUT, @XMLdoc
SELECT Chapter, ChapterNM
FROM OPENXML (@docpointer, '/Book/Chapters/Chapter',0)
WITH (Chapter int '@id',
ChapterNM varchar(50) '@name' )
GO

DECLARE @XMLdoc XML
SET @XMLdoc =
'<Book name="SQL Server 2000 Fast Answers">
<Chapters>
<Chapter id="1" name="Installation, Upgrades"/>
<Chapter id="2" name="Configuring SQL Server"/>
<Chapter id="3" name="Creating and Configuring Databases"/>
<Chapter id="4" name="SQL Server Agent and SQL Logs"/>
</Chapters>
</Book>'

EXEC dbo.usp_SEL_BookXML_Convert_To_Relational @XMLdoc

-- Storing Hierarchical Data

USE TestDB
GO
CREATE TABLE dbo.WebpageLayout
(WebpageLayoutID hierarchyid NOT NULL,
PositionDESC as WebpageLayoutID.GetLevel(),
PageURL nvarchar(50) NOT NULL)
GO

INSERT dbo.WebpageLayout
(WebpageLayoutID, PageURL)
VALUES
('/', 'http://joesack.com')

SELECT WebpageLayoutID, PositionDESC, PageURL
FROM dbo.WebpageLayout

INSERT dbo.WebpageLayout
(WebpageLayoutID, PageURL)
VALUES
('/1/', 'http://joesack.com/WordPress/')

INSERT dbo.WebpageLayout
(WebpageLayoutID, PageURL)
VALUES
('/2/', 'http://joesack.com/SQLFastTOC.htm')

SELECT WebpageLayoutID, PositionDESC, PageURL
FROM dbo.WebpageLayout

DECLARE @ParentWebpageLayoutID hierarchyid
SELECT @ParentWebpageLayoutID = CONVERT(hierarchyid, '/1/')

INSERT dbo.WebpageLayout
(WebpageLayoutID, PageURL)
VALUES
(@ParentWebpageLayoutID.GetDescendant(NULL,NULL),
'http://joesack.com/WordPress/?page_id=2')

INSERT dbo.WebpageLayout
(WebpageLayoutID, PageURL)
VALUES
(@ParentWebpageLayoutID.GetDescendant(NULL,NULL),
'http://joesack.com/WordPress/?page_id=9')

SELECT WebpageLayoutID.ToString() as WebpageLayoutID, PositionDESC, PageURL
FROM dbo.WebpageLayout

-- Returning a Specific Ancestor

DECLARE @WebpageLayoutID hierarchyid
DECLARE @New_WebpageLayoutID hierarchyid
SELECT @WebpageLayoutID = CONVERT(hierarchyid, '/1/1/')

SELECT @New_WebpageLayoutID = @WebpageLayoutID.GetAncestor( 1 )
SELECT @New_WebpageLayoutID.ToString()

-- Returning Child Nodes

DECLARE @WebpageLayoutID hierarchyid
DECLARE @New_WebpageLayoutID hierarchyid
SELECT @WebpageLayoutID = CONVERT(hierarchyid, '/1/')

SELECT @New_WebpageLayoutID = @WebpageLayoutID.GetDescendant(NULL,NULL)
SELECT @New_WebpageLayoutID.ToString()

-- Returning a Node’s Depth

DECLARE @WebpageLayoutID hierarchyid
SELECT @WebpageLayoutID = CONVERT(hierarchyid, '/1/1/1/1/')

SELECT @WebpageLayoutID.GetLevel()

-- Returning the Root Node

SELECT PageURL
FROM dbo.WebpageLayout
WHERE WebpageLayoutID = hierarchyid::GetRoot()

-- Determining Whether a Node Is a Child of the Current Node

DECLARE @WebpageLayoutID hierarchyid
SELECT @WebpageLayoutID = CONVERT(hierarchyid, '/1/')

SELECT @WebpageLayoutID.IsDescendantOf('/')
SELECT @WebpageLayoutID.IsDescendantOf('/1/1/')

-- Changing Node Locations

DECLARE @WebpageLayoutID hierarchyid
DECLARE @New_WebpageLayoutID hierarchyid
SELECT @WebpageLayoutID = CONVERT(hierarchyid, '/1/1/')

SELECT @New_WebpageLayoutID = @WebpageLayoutID.GetReparentedValue('/1/', '/2/')
SELECT @New_WebpageLayoutID.ToString()

-- Storing Spatial Data

USE TestDB
GO
CREATE TABLE dbo.MinneapolisLake
(MinneapolisLakeID int NOT NULL IDENTITY(1,1) PRIMARY KEY,
LakeNM varchar(50) NOT NULL,
LakeLocationGEOG Geography NOT NULL,
LakeLocationWKT AS LakeLocationGEOG.STAsText())
GO

-- Lake Calhoun
INSERT dbo.MinneapolisLake
(LakeNM, LakeLocationGEOG)
VALUES ('Lake Calhoun',
geography::Parse('POLYGON((-93.31593 44.94821 , -93.31924 44.93603 ,
-93.30666 44.93577 , -93.30386 44.94321 , -93.31593 44.94821 ))'))

-- Lake Harriet
INSERT dbo.MinneapolisLake
(LakeNM, LakeLocationGEOG)
VALUES ('Lake Harriet',
geography::Parse('POLYGON((-93.30776 44.92774 , -93.31379 44.91889 ,
-93.30122 44.91702 , -93.29739 44.92624 ,-93.30776 44.92774 ))'))

-- Lake of the Isles (notice several points, as this lake has an odd shape)
INSERT dbo.MinneapolisLake
(LakeNM, LakeLocationGEOG)
VALUES ('Lake of the Isles',
geography::Parse(
'POLYGON(( -93.30924 44.95847,
-93.31291 44.95360,
-93.30607 44.95178,
-93.30158 44.95543,
-93.30349 44.95689,
-93.30372 44.96261,
-93.3068 44.95720 8,
-93.30924 44.95847))'))

SELECT LakeNM, LakeLocationGEOG, LakeLocationWKT
FROM dbo.MinneapolisLake

-- Querying Spatial Data

SELECT LakeNM,
LakeLocationGEOG.STLength() Length,
LakeLocationGEOG.STArea() Area
FROM dbo.MinneapolisLake

DECLARE @DowntownStartingPoint geography
SET @DowntownStartingPoint = geography::Parse('POINT(-93.26319 44.97846)')

SELECT LakeNM,
LakeLocationGEOG.STDistance(@DowntownStartingPoint) Distance
FROM dbo.MinneapolisLake
ORDER BY LakeLocationGEOG.STDistance(@DowntownStartingPoint)

DECLARE @FlightPath geography
SET @FlightPath = geography::Parse('LINESTRING(-93.26319 44.97846, -93.30862
44.91695 )')

SELECT LakeNM,
LakeLocationGEOG.STIntersects(@FlightPath) IntersectsWithFlightPath
FROM dbo.MinneapolisLake

CREATE SPATIAL INDEX Spatial_Index_MinneapolisLake_LakeLocationGEOG
ON dbo.MinneapolisLake(LakeLocationGEOG)

DECLARE @LocationOfMyBoat geography
SET @LocationOfMyBoat= geography::Parse('POINT(-93.31329 44.94088)')

SELECT LakeNM
FROM dbo.MinneapolisLake
WHERE LakeLocationGEOG.STIntersects(@LocationOfMyBoat) = 1





