# What is Database Table?
- A database table is a collection of data organized in rows and columns.
- While the number of records, or rows, you can store in a table is limited by the amount of computer storage or disk space to which your database has access, the number of columns is limited to 1,024.

## Memory Optimized Tables
- Azure SQL also has Memory-optimized tables, tables that are stored in the database server's main memory, where rows are read and written directly to memory.
- However, there's also a physical copy of the table on disk for durability if there's a restart or disaster-recovery scenario.

## Data Types
- Some of the common data types:
  - **Character:** When you want to store character or text in the database, Azure SQL provides the nchar and nvarchar data types. Use nchar for fixed-size text data and nvarchar for variable-size text data. Using max as the length of nvarchar allows for text storage up to 2 GB in length per field in a row. Nchar and nvarchar also allow for multibyte characters that you see in languages such as Japanese and Chinese.
  - **Decimal:** Numbers with specific precision use the decimal data type. This data type must be defined by two variables. First, the precision (p), or maximum total number of decimal digits to be stored. Second, the scale(s), the number of decimal digits that are stored to the right of the decimal point.
  - **Integers:** When storing exact numbers that don't need to carry decimal values, you can use integer types. Most use cases fall into using the int data type, but there are other integer types for special cases. For small values, you can use tinyint and smallint; for large numbers, bigint is best. The money data type can be used to store currency.
  - **Bit:** The bit data type can only contain a 0 or a 1, making it perfect for boolean or true/false data.
  - **Date and Time:** Similar to the number data types, you can store dates and time in the database with various levels of precision. The date data type stores the data in the database in the YYYY-MM-DD format. If you need more accuracy, you can use the datatime2 data type, which stores the date in the YYYY-MM-DD hh:mm:ss[.nnnnnnn] format. If you just need the time, then you can use the time data type, which uses the hh:mm:ss[.nnnnnnn] format. If you're creating an application with globalization in mind, you can use the datetimeoffset data type, which contains time zone information.
  - **Binary:** If you need to store data such as images or files, you can use the binary and varbinary data types. The binary data type is for fixed-length binary data, while varbinary is for variable-length binary data.
  - **Spatial:** Azure SQL has two spatial data types; geometry and geography. The geometry type represents data in a Euclidean (flat) coordinate system, while the geography type represents data in a round-earth coordinate system. Once the data is stored in the database using these types, you can perform spatial operations with SQL, such as Nearest Neighbor queries (where is the closest pizza restaurant to my location) or locations of points in a geometrical space (where x, y, and z intersect on a graph).


## How Are The Tables Stored?
- On the file system where the database lives, tables are stored in pages. These pages are 8K files that come either as data, text/image, or index pages.
- These pages have two types of data: the actual data and the metadata about the data.
  - Metadata:
    - This header information includes the page number, page type, the amount of free space on the page, and the allocation unit ID of the object that owns the page.
- Pages store with extents, which are 8 contiguous pages. When you create a table, the first 8 pages are allocated to the table. As you add rows to the table, the table grows by adding more pages to the table.
- Extents hold eight physically contiguous pages, thus Azure SQL databases have 16 extents per megabyte. (64K per extent * 16 extents = 1,024K or 1 MB)

> **Microsoft Documentation:**
> The fundamental unit of data storage in SQL Server is the page. The disk space allocated to a data file (.mdf or .ndf) in a database is logically divided into pages numbered contiguously from 0 to n. 
> Disk I/O operations are performed at the page level. That is, SQL Server reads or writes whole data pages.
> Extents are a collection of eight physically contiguous pages and are used to efficiently manage the pages. All pages are organized into extents.

> Details: https://learn.microsoft.com/en-us/sql/relational-databases/pages-and-extents-architecture-guide

---


# Create Table
- To create a table, you use the `CREATE TABLE` statement.

## Examples

### Examples 1
First, create the cards table. There are some rules to which the card game has to adhere, and they need to be reflected in your table design:

- Only one row for each card in the cards table
- A card can have multiple language translations of the original text and card name
- The cards attributes are:
  - Card ID (a number)
  - Name (no more than 100 characters)
  - Color (no more than 10 characters)
  - Power (no card can have a power greater than 20)
  - Type (no more than 10 characters)
  - Text (Text area is 500 characters large)
  - Status
  - Art
- A card can only be one of the following colors: black, blue, green, red, white, orange
- A card can only be one of the following types: hero, monster, spell, weapon

#### Main Table
```tsql
CREATE TABLE CARDS(
    card_id INT PRIMARY KEY NOT NULL IDENTITY(1,1),
    name NVARCHAR(100) NOT NULL,
    color NVARCHAR(10) NOT NULL,
    -- CHECK is limitation for the column
    power tinyint CHECK (power <= 20),
    type NVARCHAR(10) NOT NULL,
    text NVARCHAR(500),
    status BIT NOT NULL,
    -- Varbinary is used to store binary data. In this case, it's used to store images.
    art VARBINARY(MAX)
)
```


#### Translation Table
- The translation table will store the translations of the card name and text.

```tsql
CREATE TABLE CARD_TRANSLATIONS(
    translation_id INT PRIMARY KEY NOT NULL IDENTITY(1,1),
    card_id INT NOT NULL,
    translation_card_language NVARCHAR(50) NOT NULL,
    translation_card_name NVARCHAR(500) NOT NULL,
    translation_card_text NVARCHAR(2000) NOT NULL,
);
```


#### Create The Sets Table
- The sets table will store the sets of cards that are released.

```tsql
CREATE TABLE SETS(
    set_id INT PRIMARY KEY NOT NULL IDENTITY(1,1),
    set_name NVARCHAR(50) NOT NULL,
    set_date DATE NOT NULL,
);
```


#### Create The Set Lists Table
- The set lists table will store the cards that are in each set.

```tsql
CREATE TABLE SET_LISTS(
    set_list_id INT PRIMARY KEY NOT NULL IDENTITY(1,1),
    set_id INT NOT NULL,
    card_id INT NOT NULL,
);
```











#### Constraints
- Database table constraints not only let you control what and how you store data, but define relationships between tables as well.

##### Applied Constraints
```tsql
CREATE TABLE CARDS(
    card_id INT PRIMARY KEY NOT NULL IDENTITY(1,1),
    name NVARCHAR(100) NOT NULL,
    color NVARCHAR(10) NOT NULL,
    power tinyint CHECK (power <= 20),
    type NVARCHAR(10) NOT NULL,
    text NVARCHAR(500),
    status BIT NOT NULL,
    art VARBINARY(MAX),
  -- This constraint ensures that the color column can only contain the values black, blue, green, red, white, or orange.
  -- This is a check constraint.
    CONSTRAINT CK_CARDS_COLOR CHECK (color IN ('black', 'blue', 'green', 'red', 'white', 'orange')),
    CONSTRAINT CK_CARDS_TYPE CHECK (type IN ('hero', 'monster', 'spell', 'weapon'))
);

CREATE TABLE CARD_TRANSLATIONS(
    translation_id INT PRIMARY KEY NOT NULL IDENTITY(1,1),
    card_id INT NOT NULL,
    translation_card_language NVARCHAR(50) NOT NULL,
    translation_card_name NVARCHAR(500) NOT NULL,
    translation_card_text NVARCHAR(2000) NULL,
  -- This foreign key constraint ensures that the card_id column in the CARD_TRANSLATIONS table is a foreign key that references the card_id column in the CARDS table.
    CONSTRAINT FK_CARD_TRANSLATIONS_CARDS FOREIGN KEY (card_id) REFERENCES CARDS(card_id)
);

CREATE TABLE SETS(
    set_id INT PRIMARY KEY NOT NULL IDENTITY(1,1),
    set_name NVARCHAR(50) NOT NULL,
    set_date DATE NOT NULL,
);

CREATE TABLE SET_LISTS(
    set_list_id INT PRIMARY KEY NOT NULL IDENTITY(1,1),
    set_id INT NOT NULL,
    card_id INT NOT NULL,
  -- This foreign key constraint ensures that the set_id column in the SET_LISTS table is a foreign key that references the set_id column in the SETS table.
    CONSTRAINT FK_SET_LISTS_SETS FOREIGN KEY (set_id) REFERENCES SETS(set_id),
  -- This foreign key constraint ensures that the card_id column in the SET_LISTS table is a foreign key that references the card_id column in the CARDS table.
    CONSTRAINT FK_SET_LISTS_CARDS FOREIGN KEY (card_id) REFERENCES CARDS(card_id)
);

-- We can define like this too;
-- ALTER TABLE SET_LISTS ADD CONSTRAINT FK_SET_LISTS_SETS FOREIGN KEY (set_id) REFERENCES SETS(set_id);
-- CONSTRAINT FK_SET_LISTS_CARDS FOREIGN KEY ([card_id]) REFERENCES [<namespace>].[CARDS]([card_id]);
```


# Indexes
- Indexes are used to speed up the query process in a database.
- Index Types:
  - **Clustered Index**
  - **Non-Clustered Index** 
---
- Microsoft Documentation:

A table or view can contain the following types of indexes:
- Clustered 
  - Clustered indexes sort and store the data rows in the table or view based on their key values. 
  - These key values are the columns included in the index definition. 
  - There can be only one clustered index per table, because the data rows themselves can be stored in only one order.
  - The only time the data rows in a table are stored in sorted order is when the table contains a clustered index. 
  - When a table has a clustered index, the table is called a clustered table. 
  - If a table has no clustered index, its data rows are stored in an unordered structure called a heap.

- Nonclustered 
  - Nonclustered indexes have a structure separate from the data rows. 
  - A nonclustered index contains the nonclustered index key values and each key value entry has a pointer to the data row that contains the key value. 
  - The pointer from an index row in a nonclustered index to a data row is called a row locator. 
  - The structure of the row locator depends on whether the data pages are stored in a heap or a clustered table. 
  - **For a heap, a row locator is a pointer to the row. For a clustered table, the row locator is the clustered index key.**
  - You can add nonkey columns to the leaf level of the nonclustered index to by-pass existing index key limits, and execute fully covered, indexed, queries.

> For Limits: https://learn.microsoft.com/en-us/sql/sql-server/maximum-capacity-specifications-for-sql-server?view=sql-server-ver16

>  Creation With Multiple Columns: https://learn.microsoft.com/en-us/sql/relational-databases/indexes/create-indexes-with-included-columns?view=sql-server-ver16

> SQL Server create indexes automatically when you create a PRIMARY KEY or UNIQUE constraint.

> Other Index Details: https://learn.microsoft.com/en-us/sql/relational-databases/indexes/indexes?view=sql-server-ver16
> https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide?view=sql-server-ver16

> Query Optimizer And Indexes: https://learn.microsoft.com/en-us/sql/relational-databases/indexes/indexes?view=sql-server-ver16

## Full-Text Indexes
- Full-text indexes are used to search large amounts of text data quickly.
- We can apply the full-text index to columns that have any of the following data types: `char`, `varchar`, `nchar`, `nvarchar`, `text`, `ntext`, `image`, `xml`, or `varbinary(max)` and FILESTREAM.

> Full-Text Index Details: https://learn.microsoft.com/en-us/sql/relational-databases/search/populate-full-text-indexes?view=sql-server-ver16

## Scan Types
- **Index Seek:** The query optimizer searches for the data in particular index pages and retrieves the data.
- **Index Scan:** The query optimizer searches the entire index for the data.
- **Table Scan:** The query optimizer searches the entire table for the data.