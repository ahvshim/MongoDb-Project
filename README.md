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

2.  List all the customers that their annual income is less than 20,000 and bought products in 2015.

```mongo
adventureworks.sales.aggregate([
  {
    $lookup: {
      from: "products",
      localField: "ProductKey",
      foreignField: "ProductKey",
      as: "product_info",
    },
  },
  {
    $unwind: "$product_info",
  },
  {
    $lookup: {
      from: "customers",
      localField: "CustomerKey",
      foreignField: "CustomerKey",
      as: "customer_info",
    },
  },
  {
    $unwind: "$customer_info",
  },
  {
    $match: {
      "customer_info.AnnualIncome": {
        $lt: 20000,
      },
    },
  },
  {
    $addFields: {
      OrderDate: {
        $toDate: "$OrderDate",
      },
    },
  },
  {
    $project: {
      FirstName: "$customer_info.FirstName",
      LastName: "$customer_info.LastName",
      AnnualIncome: "$customer_info.AnnualIncome",
      ProductName: "$product_info.ProductName",
      Year: {
        $year: "$OrderDate",
      },
    },
  },
])
```
<br>

![query 2](https://user-images.githubusercontent.com/126220185/222957659-4e238eff-a365-4a43-bbc0-b73ccb484e4d.png)

<br>
3.  List all customers and their order quantities in the year 2017

```mongo
adventureworks.sales.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "CustomerKey",
      foreignField: "CustomerKey",
      as: "customer_info",
    },
  },
  {
    $lookup: {
      from: "products",
      localField: "ProductKey",
      foreignField: "ProductKey",
      as: "product_info",
    },
  },
  {
    $unwind: "$customer_info",
  },
  {
    $unwind: "$product_info",
  },
  {
    $addFields: {
      OrderDate: {
        $toDate: "$OrderDate",
      },
    },
  },
  {
    $match: {
      OrderDate: {
        $gte: ISODate("2017-01-01"),
        $lt: ISODate("2018-01-01"),
      },
    },
  },
  {
    $group: {
      _id: "$customer_info.CustomerKey",
      FirstName: {
        $first: "$customer_info.FirstName",
      },
      LastName: {
        $first: "$customer_info.LastName",
      },
      ProductName: {
        $first: "$product_info.ProductName",
      },
      OrderQuantity: {
        $sum: "$OrderQuantity",
      },
      OrderDate: {
        $first: "$OrderDate",
      },
    },
  },
  { $sort: { OrderQuantity: -1 } },
  {
    $project: {
      FirstName: 1,
      LastName: 1,
      ProductName: 1,
      OrderQuantity: 1,
      Year: {
        $year: "$OrderDate",
      },
      _id: 0,
    },
  },
])
```
<br>

![query 3](https://user-images.githubusercontent.com/126220185/222960493-b026cda2-56e2-42ad-bdbb-e7e911c1fd30.png)

4.  Count the products that purchased the same item in all years.

```mongo
adventureworks.sales.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "CustomerKey",
      foreignField: "CustomerKey",
      as: "customer_info",
    },
  },
  {
    $lookup: {
      from: "products",
      localField: "ProductKey",
      foreignField: "ProductKey",
      as: "product_info",
    },
  },
  {
    $unwind: "$customer_info",
  },
  {
    $unwind: "$product_info",
  },
  {
    $group: {
      _id: "$product_info.ProductName",
      quantity_sold: {
        $sum: "$OrderQuantity",
      },
    },
  },
  {
    $project: {
      _id: 0,
      ProductName: "$_id",
      quantity_sold: 1,
    },
  },
  {
    $sort: {
      quantity_sold: -1,
    },
  },
])
```
<br>

![query 4](https://user-images.githubusercontent.com/126220185/222960581-b417ac41-fadb-442f-88a7-ad8141ab91f1.png)

5.  Count the returned products group by region.

```mongo
adventureworks.returns.aggregate([
  {
    $lookup: {
      from: "territories",
      localField: "TerritoryKey",
      foreignField: "TerritoryKey",
      as: "territory_info",
    },
  },
  {
    $unwind: "$territory_info",
  },
  {
    $group: {
      _id: "$territory_info.Region",
      Total_Return: {
        $sum: 1,
      },
    },
  },
  {
    $sort: {
      Total_Return: -1,
    },
  },
  {
    $project: {
      Total_Return: 1,
      Region: "$_id",
      _id: 0,
    },
  },
])
```
<br>

![query 5](https://user-images.githubusercontent.com/126220185/222960651-36176685-f0e2-4af7-8b81-417d61f39414.png)

6.   Find out the profit of the top 5 products for 2017.

```mongo
adventureworks.sales.aggregate([
  {
    $addFields: {
      OrderDate: {
        $toDate: "$OrderDate",
      },
    },
  },
  {
    $match: {
      OrderDate: {
        $gte: Date("2017-01-01"),
        $lt: Date("2018-01-01"),
      },
    },
  },
  {
    $lookup: {
      from: "products",
      localField: "ProductKey",
      foreignField: "ProductKey",
      as: "product_info",
    },
  },
  {
    $unwind: "$product_info",
  },
  {
    $addFields: {
      Profit: {
        $subtract: [
          "$product_info.ProductPrice",
          "$product_info.ProductCost",
        ],
      },
    },
  },
  {
    $project: {
      ProductKey: "$product_info.ProductKey",
      ProductName: "$product_info.ProductName",
      ProductCost: "$product_info.ProductCost",
      ProductPrice: "$product_info.ProductPrice",
      Profit: 1,
      Year: {
        $year: "$OrderDate",
      },
      _id: 0,
    },
  },
  {
    $limit: 5,
  },
])
```
<br>

![query 6](https://user-images.githubusercontent.com/126220185/222960778-0253493f-ff16-4190-a23a-b409c3b08435.png)

7.  Find the total returns in each year (2015, 2016, 2017)

```mongo
adventureworks.returns.aggregate([
  {
    $addFields: {
      ReturnDate: {
        $toDate: "$ReturnDate",
      },
    },
  },
  {
    $facet: {
      year_2015: [
        {
          $match: {
            ReturnDate: {
              $gte: new Date("2015-01-01"),
              $lte: new Date("2015-12-31"),
            },
          },
        },
        {
          $group: {
            _id: null,
            Total_Returns: {
              $sum: "$ReturnQuantity",
            },
          },
        },
        {
          $project: {
            Year: {
              $literal: "2015",
            },
            Total_Returns: 1,
            _id: 0,
          },
        },
      ],
      year_2016: [
        {
          $match: {
            ReturnDate: {
              $gte: new Date("2016-01-01"),
              $lte: new Date("2016-12-31"),
            },
          },
        },
        {
          $group: {
            _id: null,
            Total_Returns: {
              $sum: "$ReturnQuantity",
            },
          },
        },
        {
          $project: {
            Year: {
              $literal: "2016",
            },
            Total_Returns: 1,
            _id: 0,
          },
        },
      ],
      year_2017: [
        {
          $match: {
            ReturnDate: {
              $gte: new Date("2017-01-01"),
              $lte: new Date("2017-12-31"),
            },
          },
        },
        {
          $group: {
            _id: null,
            Total_Returns: {
              $sum: "$ReturnQuantity",
            },
          },
        },
        {
          $project: {
            Year: {
              $literal: "2017",
            },
            Total_Returns: 1,
            _id: 0,
          },
        },
      ],
    },
  },
  {
    $project: {
      results: {
        $concatArrays: [
          "$year_2015",
          "$year_2016",
          "$year_2017",
        ],
      },
    },
  },
  {
    $unwind: "$results",
  },
  {
    $replaceRoot: {
      newRoot: "$results",
    },
  },
])
```

<br>

![query 7](https://user-images.githubusercontent.com/126220185/222960858-0ea95966-e543-4827-b287-cfc26094167f.png)

<br>

# Data-Models Discussion

<div align="justify">
Relational data models such as MySQL workbench are good at storing 
structured and related data, where data is organized into tables with relationships 
defined between them. They provide a number of advantages, such as enforcing data 
integrity and consistency through foreign keys, and enabling complex queries and 
transactions.
</div>
<br>
<div align="justify">
On the other hand, document-based data models, such as MongoDB compass, 
store data in semi-structured or unstructured format, in the form of documents (key-value pairs),
which can be nested and embedded. These data models are more flexible 
and scalable, as they can store any kind of data without having to define a fixed schema 
beforehand, and can handle large amounts of unstructured data.
</div>
<br>
<div align="justify">
When considering which model is more applicable, it's important to think about 
the specific needs of the case study. In our case, the dataset “Adventure Works” is 
highly structured, with well-defined relationships thus, a relational SQL data model 
may be a better choice.
</div>
<br>
<div align="justify">
In summary, the choice between a relational and a document-based data model 
should be based on the specific needs of the case study, and the strengths and 
limitations of each model should be taken into account. Relational data model is 
advisable to dataset’s that are structured and well defined relationships same to this 
case study, while data that is semi-structured or unstructured, and requires scalability 
and flexibility, a document-based data model may be a better option.
</div>


