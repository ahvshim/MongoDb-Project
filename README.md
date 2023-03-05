# Executive Summary
# Introduction
<div align="justify">
After completed the <a href="https://github.com/ahvshim/SQL-Project" target="_blank">Adventure works relational database</a> 
in previous project,and applied SQL queries based on a given scenario that was proposed,
now we’d like to develop a 
No-SQL document-based database using MongoDB Compass application and apply 
similar aggregations. <br>
The very first step was exporting our tables as a JSON format from 
‘adventureworks’ database stored in MySQL Workbench graphical tool.
</div>

#  Document design
<div align="justify">
The choice between embedded and referenced document design in MongoDB 
compass would depend on the relationships between the tables and the data access 
patterns in our application. 
</div>
<br>
<div align="justify">
In general, embedded documents are useful for modeling one-to-many 
relationships where the related data will always be retrieved together with the parent 
document. While reference design is useful for modeling many-to-many relationships, 
where the related data is not always needed. In such cases, references (ObjectIDs) are 
used to link documents.
</div>
<br>
<div align="justify">
However, in our case as we using the AdventureWork database from SQL, it 
seems like a combination of both embedded and referenced document design would 
be appropriate, based on the relationships between our tables and the data access 
patterns in our application.
</div>
<br>
<div align="justify">
The tables "sales_2015", "sales_2016" and "sales_2017" have been stored as 
separate documents in a "sales" collection and referenced from the "customers" and 
"products" collections. 
</div>
<br>
<div align="justify">
The tables “products_categories”, “products_subcategories” and “products” 
were embedded in a separate document in “products” collection. <br>
The "returns", “customers” and "territories" tables have been stored as a 
separate collections and referenced from the "sales" collection. While the table 
“calendar” was excluded as now we have the dates in sales collection.
</div>

However, as we had ten different tables in SQL database, now we only have 
five collections in our document-based database named “adventureworks” that 
involves our nine tables we discussed above. These collections can be summarized in 
the Figure 1. Below
<br>
![collections](https://user-images.githubusercontent.com/126220185/222956520-7030b1ed-d8e7-4755-a5d0-ec11166853cb.png)

# Aggregation Pipeline Code
<div align="justify">
While SQL queries are written in Structured Query Language (SQL), which is 
used for relational databases. MongoDB is a NoSQL database and uses a different 
query language, MongoDB Query Language (MQL).
</div>
<br>
<div align="justify">
Before obtaining any aggregation stages, we created indexes on fields that we 
will frequently search, sort, or group by within our collections. By creating an index 
on these fields, MongoDB will use the index to locate the relevant documents for the 
query, which will result in faster query execution times. As we are using similar 
queries to those in SQL project, these fields are equivalent to Primary Keys.
</div>
<br>
1.  Find all the products profit and identify them by their names in 
ascending order

```mongo
adventureworks.products.aggregate([
  {
    $project: {
      ProductName: 1,
      ProductCost: 1,
      ProductPrice: 1,
      Profit: {
        $subtract: [
          "$ProductPrice",
          "$ProductCost",
        ],
      },
    },
  },
  {
    $sort: {
      Profit: -1,
    },
  },
])
```
<br>
![query 1](https://user-images.githubusercontent.com/126220185/222957434-fe62aa35-53bf-484e-a65e-b8d51a6bf3db.png)
