# SQL
## Constraints
* PK: 
  1. A table can have only primary key 
  2. not accept NULL values
* FK:
* Index:
* Unique: 
  1. contain NULL value, 
  2.May have multiple in same table
  3 prevent duplicate values in certain column 
* Composition: multiple columns uniquely identifies each record
### Tables and views
* different 
  1. view is created by collection data from certain SQL
  2. Table physical existing, view is not
  3. tables will not accpet modification
### Cursors
> A pointer to a specific row within a query result
* mainly use for data manipulation 
### Triggers
* use for notification
* when database modified,notify other in same system
### Stored Procedures
* a set of Structured Query Language (SQL) statements with an assigned name save as a group in DMS
Functions
### Functions

# JDBK
> An API that allows Java applications to connect to a relational database

## Features
* no need to develop different code for different databases

## JDBC Architecture

### Drivers
> provide connection to a database
* JDCBC-ODBC Bridge
* Native JDBC Driver
* All Java JDBC Net Driver
* Native Protocol All Java JDBC

## Package
* `java.sql`
* `javax.sql`

## Development Process
Source(.java)-->Class(byte code)-->JDBC-->ODBC--->DB
1. Get a connection to database
2. Create a Statement object
3. Execute SQL query
4. Process Result set
### 1. Load JDBC driver
* Initialize a driver in order to open a channel for DB
```
Class.forName("com.mysql.cj.jdbc.Driver")
```
### 2. Connection(`hava.saql`)
#### DriverManager
* getConnection("URL","USERNAME","PASSWORD")
```
Connection cn =DrvierManager.getConnection(url,usder,password)
```
* `java.util.Properties`:read and write a configuration file
```
Properties props= new Properties();
props.load(new FileInputeStream("file"));
String url=props.getProerty("url_name");
String user=props.getProerty("user_name");
String password=props.getProerty("password_name");
Connection cn =DrvierManager.getConnection(url,usder,password)
```
### 2. Create a Statement
* Statment `createStatement()`
```
Statement myStatement=cn.createStatement();
```
#### Prepared Statement
> a precomplied SQL statement
* Instead of complete SQL, Prepared statement can set parameter placeholders by `?` in order to reuse same structure of SQL with different value
* It can use in `executeQuery()` and `executeUpdate()`
* PreparedStatment `prepareStatment()`
  * `setString()`
  * `setInt()`
  * `setDouble()`  
```
PreparedStatement myStatement=cn.prepareStatement("selec* from table_name"+"where columname>? and columname=?");
myStatement=set.Double();
myStatement=set.String();
ResultSet res=myStatement.executeQuery();
```
#### Stored Procedures
> A group of SQL statements that perform a particular task

* Callable statement `prepareCall()`: to call stored procedures from java
  * `setString()`
  * `setInt()`
  * `setDouble()`
  * `registerOutParameter()`
  * `getInt()
```
myCall=cn.perpareCall("{call some_stored_proc()}");
mycall.set_data_type()(index,new_value)
myCall.execute()
```
* INOUT parameter
  1. use `?` as placeholder
  2. register the parameter as outer paramter(need to specify type)
```
myCall=cn.perpareCall("{call some_stored_proc(?)}");
mycall.registerOutParameter(1,Types.VARCHAR);
myCall.setString(1,theDepartment);
String res=myCall.getString(1); retreive the value of out parameter
```
* OUT parameters
```
myCall=cn.perpareCall("{call some_stored_proc(?,?)}");
myCall.setString(1,value);
mycall.registerOutParameter(2,Types.VARCHAR);# USE THIS FOR INOUT
String res=myCall.getString(1); retreive the value of out parameter
```
* Result set: store all results
```
myCall=cn.perpareCall("{call some_stored_proc(?)}");
myCall.execute();
res=myCall.getResultset()
```
### 3. Excute SQL query
```
ResultSet res=myStatement.execute("SQL")
```
* boolean `execute()` - DDL
* int `executeUpdate()` - DML
* ResultSet `executeQuery()` - `Select`

####  DML Operation
* use `executeUpdate()` function, result will be the number(`int`) of changed row
```
int rowUpdate=myStatement.executeUpdate("insert into table_name" +"(column names)"+"values"+("each column data")// INSERT
int rowUpdate=myStatement.executeUpdate("update table_name" +"set column_name=new_value"+"where given_row")//UPDATE
int rowUpdate=myStatement.executeUpdate("delete table_name" +"where given_row")//DELETE
```
### 4. Process the Result set
* Result set is initially placed before first row
* Method: ResultSet.next()
  1. move forward one row
  2. return `true` if there are "unscanned" rows 
```
 while (res.next()){
      //read data from each row
 }
```
### 5. Retreive ResultSet
* Collection of methods for reading data
```
res.getString("column name" or "column index");
```
#### Methods
* `getString()`
* `getDate()`
* `getInt()`
* `getObject()`
* `getDouble()`
* `getMetaData()`
#### Types: 
  * INTEGER,VARCHAR,DATE,DOUBLE
#### MetaData
##### DataBase MetaData
* Access database information
* DatabaseMetaData methods
  - `getDatabaseProductName()
  - `getDatabaseProductVersion()`
  - `getDriverName()`
```
DatabaseMetaData databaseMetaDdata=cn.getMetaData();
```
* Access database table information
```
res= databaseMetaData.getTables();
columns=databaseMetaData.getColumns();
```
#####  ResultSet MetaData
* getColumnName()
* getColumnType()
* getPrecision()
* isCurrency()
```
ResultSetMetaData MetaData= res.getMetaData();
int cc=MetaData.get ColumnCount()
```
#### BLOBS and CLOBs
> binaray large object is a collection of binary data stored as a single entity in a db.
* Blobs are typically images, audio or other mutilmedia objects

#### Write BLOBS
```
PreparedStatement stmt=cn.prepareStatement(sql);
File newfile=new File("file");
FileInputStream input=new FileInputStrea(newfile);
stmt.setBinaryStream(1,input);#Update DB with binary data
```
#### Read BLOBS
```
res=stmt.executeQuery(sql);# select BLOB objects
File newfile=new File("file_name");
output =new FileOutputStrea(newfile);
if(stmt.next()){
    input=res.getBinaryStream("BLOB_name")}
    byte[] buffer=new byte[1024];
    while (input.read(buffer))>0{
      output.write(buffer);
      }
     Syste.out.println("\nSaved to file:"+theFile.getAbsolutePath())
}
```
####  CLOBS
> Character large object is a collection of character data in a database management system
* typically used to store large text documents（plain text or XML）
* Write
```
stmt.setCharacterStream(1,input);#Update DB with CLOB data
```
* Read
```
Reader input =res.getCharacterStream("file")
```
### `javax.sql`

### Batch Updates

### Transactions
> A unit of one or more statements
* statement executed together(all or not)
* By default, the database connection is to auto-commit
```
cn.setAutoCommit(false);
myStatement=cn.createStateme t();
myStatement.executeUpdate("");
myStatement.executeUpdate("");
boolean ok=askUserIfOkToSave();
if (ok){
  cn.commit()}
else{
  cn.rollback();}
```

### Scrollable ResultSet
