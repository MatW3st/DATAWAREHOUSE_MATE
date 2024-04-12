## PRACTICA PARA HACER UNA DATA WAREHOUSE
- Juan Mateo Hernandez de Luna | 5TIDSM-G1

##### EJERCICIO 1 Crear una base de datos de almacén de datos y una secuencia 
- 1. Inicie SSMS y conéctese a su instancia de SQL Server. Abra una nueva ventana de consulta  haciendo clic en el botón Nueva consulta. 
- 2. Desde el contexto de la base de datos maestra, cree una nueva base de datos llamada  tk463DW. Antes de crear la base de datos, compruebe si existe y suéltela si es necesario.  Siempre debe verificar si existe un objeto y soltarlo si es necesario. La base de datos debe  tener las siguientes propiedades: 

```sql
-- punto 1  
USE master; 
IF DB_ID('TK463DW') IS NOT NULL 
DROP DATABASE TK463DW; 
GO 
```
<img src="/CAP1.png" alt="I" width="auto" height="auto" >

```sql
--PUNTO 2
CREATE DATABASE TK463DW 
ON PRIMARY 
(NAME = N'TK463DW', FILENAME = N'C:\TK463\TK463DW.mdf', 
SIZE = 307200KB , FILEGROWTH = 10240KB ) 
LOG ON 
(NAME = N'TK463DW_log', FILENAME = N'C:\TK463\TK463DW_log.ldf', 
SIZE = 51200KB , FILEGROWTH = 10%); 
GO 
```
- 3. Después de crear la base de datos, cambie el modelo de recuperación a Simple. Aquí está  el código completo de creación de la base de datos. 

```sql
ALTER DATABASE TK463DW SET RECOVERY SIMPLE WITH NO_WAIT; 
GO 
```

- 4. En su nuevo almacén de datos, cree un objeto de secuencia. Nómbrelo  seqcustomerDwkey. Comience a numerar con 1 y use un incremento de 1. Para otras  
opciones de secuencia, use los valores predeterminados de SQL Server. Puedes usar el  siguiente código. 

```sql
USE TK463DW; 
GO 
IF OBJECT_ID('dbo.SeqCustomerDwKey','SO') IS NOT NULL 
DROP SEQUENCE dbo.SeqCustomerDwKey; 
GO 
CREATE SEQUENCE dbo.SeqCustomerDwKey AS INT 
START WITH 1 
INCREMENT BY 1; 
GO 
```

#### Ejercicio 2. Creando Dimensiones 
- En este ejercicio, creará la dimensión Clientes, para lo cual tendrá que implementar muchos de los  conocimientos adquiridos en este capítulo y en el anterior. En la base de datos Adventure  WorksDW2012, la dimensión DimCustomer, que servirá como fuente para la dimensión  Customers, está parcialmente cubierta de nieve. Tiene una tabla de búsqueda de un nivel llamada  DimGeography. Desnormalizarás completamente esta dimensión. Además, agregará las columnas  necesarias para admitir una dimensión SCD Tipo 2 y un par de columnas calculadas. Además de la  dimensión Clientes, creará las dimensiones Productos y Fechas.

- 1. Crear la Dimensión Customers. El origen de esta dimensión es la dimensión DimCustomer de la base de datos de ejemplo AdventureWorksDW2012.
```sql
create database AdventureWorksDW2012
use AdventureWorksDW2012

CREATE TABLE dbo.Customers 
( 
CustomerDwKey INT NOT NULL, 
CustomerKey INT NOT NULL, 
FullName NVARCHAR(150) NULL, 
EmailAddress NVARCHAR(50) NULL, 
BirthDate DATE NULL, 
MaritalStatus NCHAR(1) NULL, 
Gender NCHAR(1) NULL, 
Education NVARCHAR(40) NULL, 
Occupation NVARCHAR(100) NULL, 
City NVARCHAR(30) NULL, 
StateProvince NVARCHAR(50) NULL, 
CountryRegion NVARCHAR(50) NULL, 
Age AS 
CASE 
WHEN DATEDIFF(yy, BirthDate, CURRENT_TIMESTAMP) <= 40 
THEN 'Younger' 
WHEN DATEDIFF(yy, BirthDate, CURRENT_TIMESTAMP) > 50 
THEN 'Older' 
ELSE 'Middle Age' 
END,
CurrentFlag BIT NOT NULL DEFAULT 1, 
CONSTRAINT PK_Customers PRIMARY KEY (CustomerDwKey) 
); 
GO 
select * from dbo.Customers
```

- 2. Cree la dimensión Productos. El origen de esta dimensión es la dimensión DimProducts de  la base de datos de ejemplo AdventureWorksDW2012. Utilice la Tabla 2-2 para obtener la  información que necesita para crear y completar esta tabla. 
```sql
CREATE TABLE dbo.Dates 
( 
DateKey INT NOT NULL, 
FullDate DATE NOT NULL, 
MonthNumberName NVARCHAR(15) NULL, 
CalendarQuarter TINYINT NULL, 
CalendarYear SMALLINT NULL, 
CONSTRAINT PK_Dates PRIMARY KEY (DateKey) 
);
GO
select * from dbo.Dates 
```
<img src="/CAP3.png" alt="I" width="auto" height="auto" >


- 1. Su código para crear la tabla de hechos InternetSales debe ser similar al código de la  siguiente lista. 
```sql
CREATE TABLE dbo.InternetSales 
( 
InternetSalesKey INT NOT NULL IDENTITY(1,1), 
CustomerDwKey INT NOT NULL, 
ProductKey INT NOT NULL, 
DateKey INT NOT NULL, 
OrderQuantity SMALLINT NOT NULL DEFAULT 0, 
SalesAmount MONEY NOT NULL DEFAULT 0, 
UnitPrice MONEY NOT NULL DEFAULT 0, 
DiscountAmount FLOAT NOT NULL DEFAULT 0,
CONSTRAINT PK_InternetSales 
PRIMARY KEY (InternetSalesKey) 
); 
GO 
```
<img src="/CAP4.png" alt="I" width="auto" height="auto" >

- 2. Modifique la tabla de hechos InternetSales para agregar restricciones de clave externa para  las relaciones con las tres dimensiones. El código se muestra en el siguiente listado.
```sql
--ALTERADO
ALTER TABLE dbo.InternetSales ADD CONSTRAINT
FK_InternetSales_Customers_New FOREIGN KEY(CustomerDwKey)
REFERENCES dbo.Customers (CustomerDwKey);
ALTER TABLE dbo.InternetSales ADD CONSTRAINT
FK_InternetSales_Products_New FOREIGN KEY(ProductKey)
REFERENCES dbo.Products (ProductKey);
ALTER TABLE dbo.InternetSales ADD CONSTRAINT
FK_InternetSales_Dates_New FOREIGN KEY(DateKey)
REFERENCES dbo.Dates (DateKey);
GO

```
<img src="/CAP5.png" alt="I" width="auto" height="auto" >

##### 3. Cree un diagrama de base de datos, como se muestra en la Figura 2-1. Nómbrelo  internetsalesDW y guárdelo.

<img src="/CAP6.png" alt="I" width="auto" height="auto" >

