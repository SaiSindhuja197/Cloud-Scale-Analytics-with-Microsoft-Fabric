# Lab 03: Analyze data with Apache Spark SDP -Fabric

## Lab Overview

The primary objective of the lab is to analyze data using Apache Spark within the Microsoft Fabric environment. The tasks include creating a notebook, loading data into a Spark DataFrame, exploring the data, transforming it, and visualizing insights.

## Lab Objectives
Task 1 : Create a notebook<br>
Task 2 : Load data into a dataframe<br>
Task 3 : Explore data in a dataframe<br>
Task 4 : Use Spark to transform data files<br>
Task 5 : Visualize data with Spark<br>


### Estimated timing: 45 minutes

## Prerequisites: 
  
### Create a lakehouse and upload files

   Now that you have a workspace, it's time to switch to the *Data engineering* experience in the portal and create a data lakehouse for the data files you're going to analyze.

1. At the bottom left of the Power BI portal, select the **Power BI** icon and switch to the **Data Engineering** experience.

2. In the **Synapse Data Engineering** home page, choose the Existing **Lakehouse_<inject key="DeploymentID" enableCopy="false"/>**.

3. Return to the web browser tab containing your lakehouse, and In the menu select **ellipse** icon for the **Files** folder in the **Explorer** pane, select **Upload** and **Upload folder**, and then upload the **orders** folder from **C:\LabFiles\Files\orders** to the lakehouse.

4. After the files have been uploaded, expand **Files** and select the **orders** folder; and verify that the CSV files have been uploaded, as shown here:


   ![Screenshot of uploaded files in a lakehouse.](./Images/uploaded-files1.png)

## Architecture Diagram

  ![Navigate-To-AAD](./Images/ws/lab_03.1.png)


## Task 1 : Create a notebook

To work with data in Apache Spark, you can create a *notebook*. Notebooks provide an interactive environment in which you can write and run code (in multiple languages), and add notes to document it.

1. On the **Home** page while viewing the contents of the **orders** folder in your datalake, in the **Open notebook (1)** menu, select **New notebook (2)**.

    ![](./Images/Pg7-Notebook-S1.png)

    > After a few seconds, a new notebook containing a single *cell* will open. Notebooks are made up of one or more cells that can contain *code* or *markdown* (formatted text).
   

3. Select the first cell (which is currently a *code* cell), and then in the dynamic tool bar at its top-right, use the **M&#8595;** button to convert the cell to a *markdown* cell.

    When the cell changes to a markdown cell, the text it contains is rendered.

4. Use the **&#128393;** (Edit) button to switch the cell to editing mode, then modify the markdown as follows:

    ```
   # Sales order data exploration

   Use the code in this notebook to explore sales order data.
    ```

5. Click anywhere in the notebook outside of the cell to stop editing it and see the rendered markdown.

## Task 2 : Load data into a dataframe

  Now you're ready to run code that loads the data into a *dataframe*. Dataframes in Spark are similar to Pandas dataframes in Python, and provide a common structure for 
  working with data in rows and columns.

   > **Note**: Spark supports multiple coding languages, including Scala, Java, and others. In this exercise, we'll use *PySpark*, which is a Spark-optimized variant of Python. 
             PySpark is one of the most commonly used languages on Spark and is the default language in Fabric notebooks.

1. With the notebook visible, expand the **Files** list and select the **orders** folder so that the CSV files are listed next to the notebook editor, like this:

      ![Screenshot of a notebook with a Files pane.](./Images/notebook-files1.png)

2. In the menu select **ellipse** icon for **2019.csv**, select **Load data** > **Spark**.

      ![](./Images/Pg7-LoadData-S2.png)

3. A new code cell containing the following code should be added to the notebook:

    ```python
   df = spark.read.format("csv").option("header","true").load("Files/orders/2019.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/orders/2019.csv".
   display(df)
    ```

    > **Tip**: You can hide the Lakehouse explorer panes on the left by using their **<<** icons. Doing so will help you focus on the notebook.

4. Use the **&#9655; Run cell** button on the left of the cell to run it.

    > **Note**: Since this is the first time you've run any Spark code, a Spark session must be started. This means that the first run in the session can take a minute or so to complete. Subsequent runs will be quicker.

5. When the cell command has completed, review the output below the cell, which should look similar to this:

    | Index | SO43701 | 11 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com | Mountain-100 Silver, 44 | 16 | 3399.99 | 271.9992 |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO43704 | 1 | 2019-07-01 | Julio Ruiz | julio1@adventure-works.com | Mountain-100 Black, 48 | 1 | 3374.99 | 269.9992 |
    | 2 | SO43705 | 1 | 2019-07-01 | Curtis Lu | curtis9@adventure-works.com | Mountain-100 Silver, 38 | 1 | 3399.99 | 271.9992 |
    | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse |

    The output shows the rows and columns of data from the 2019.csv file. However, note that the column headers don't look right. The default code used to load the data into a dataframe assumes that the CSV file includes the column names in the first row, but in this case the CSV file just includes the data with no header information.

6. Modify the code to set the **header** option to **false** as follows:

    ```python
   df = spark.read.format("csv").option("header","false").load("Files/orders/2019.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/orders/2019.csv".
   display(df)
    ```

7. Re-run the cell and review the output, which should look similar to this:

   | Index | _c0 | _c1 | _c2 | _c3 | _c4 | _c5 | _c6 | _c7 | _c8 |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO43701 | 11 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com | Mountain-100 Silver, 44 | 16 | 3399.99 | 271.9992 |
    | 2 | SO43704 | 1 | 2019-07-01 | Julio Ruiz | julio1@adventure-works.com | Mountain-100 Black, 48 | 1 | 3374.99 | 269.9992 |
    | 3 | SO43705 | 1 | 2019-07-01 | Curtis Lu | curtis9@adventure-works.com | Mountain-100 Silver, 38 | 1 | 3399.99 | 271.9992 |
    | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse |

    Now the dataframe correctly includes first row as data values, but the column names are auto-generated and not very helpful. To make sense of the data, you need to explicitly define the correct schema and data type for the data values in the file.

8. Modify the code as follows to define a schema and apply it when loading the data:

    ```python
   from pyspark.sql.types import *

   orderSchema = StructType([
       StructField("SalesOrderNumber", StringType()),
       StructField("SalesOrderLineNumber", IntegerType()),
       StructField("OrderDate", DateType()),
       StructField("CustomerName", StringType()),
       StructField("Email", StringType()),
       StructField("Item", StringType()),
       StructField("Quantity", IntegerType()),
       StructField("UnitPrice", FloatType()),
       StructField("Tax", FloatType())
       ])

   df = spark.read.format("csv").schema(orderSchema).load("Files/orders/2019.csv")
   display(df)
    ```

9. Run the modified cell and review the output, which should look similar to this:

   | Index | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | Email | Item | Quantity | UnitPrice | Tax |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | 1 | SO43701 | 11 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com | Mountain-100 Silver, 44 | 16 | 3399.99 | 271.9992 |
    | 2 | SO43704 | 1 | 2019-07-01 | Julio Ruiz | julio1@adventure-works.com | Mountain-100 Black, 48 | 1 | 3374.99 | 269.9992 |
    | 3 | SO43705 | 1 | 2019-07-01 | Curtis Lu | curtis9@adventure-works.com | Mountain-100 Silver, 38 | 1 | 3399.99 | 271.9992 |
    | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse | ellipse |

    Now the dataframe includes the correct column names (in addition to the **Index**, which is a built-in column in all dataframes based on the ordinal position of each row). The data types of the columns are specified using a standard set of types defined in the Spark SQL library, which were imported at the beginning of the cell.

10. Confirm that your changes have been applied to the data by viewing the dataframe. Run the following cell:

    ```python
    display(df)
    ```

11. The dataframe includes only the data from the **2019.csv** file. Modify the code so that the file path uses a \* wildcard to read the sales order data from all of the files in the **orders** folder:

       ```python
       from pyspark.sql.types import *
    
       orderSchema = StructType([
         StructField("SalesOrderNumber", StringType()),
         StructField("SalesOrderLineNumber", IntegerType()),
         StructField("OrderDate", DateType()),
         StructField("CustomerName", StringType()),
         StructField("Email", StringType()),
         StructField("Item", StringType()),
         StructField("Quantity", IntegerType()),
         StructField("UnitPrice", FloatType()),
         StructField("Tax", FloatType())
    ])
    
      df = spark.read.format("csv").schema(orderSchema).load("Files/orders/*.csv")
      display(df)
      ```

12. Run the modified code cell and review the output, which should now include sales for 2019, 2020, and 2021.

    **Note**: Only a subset of the rows is displayed, so you may not be able to see examples from all years.

## Task 3 : Explore data in a dataframe

The dataframe object includes a wide range of functions that you can use to filter, group, and otherwise manipulate the data it contains.

1. Use the **+ Code** icon below the cell output to add a new code cell to the notebook, and enter the following code in it.

    ```Python
   customers = df['CustomerName', 'Email']
   print(customers.count())
   print(customers.distinct().count())
   display(customers.distinct())
    ```

1. Run the new code cell, and review the results. Observe the following details:
    - When you perform an operation on a dataframe, the result is a new dataframe (in this case, a new **customers** dataframe is created by selecting a specific subset of 
       columns from the **df** dataframe)
    - Dataframes provide functions such as **count** and **distinct** that can be used to summarize and filter the data they contain.
    - The `dataframe['Field1', 'Field2', ellipse]` syntax is a shorthand way of defining a subset of columns. You can also use **select** method, so the first line of the code above could be written as `customers = df.select("CustomerName", "Email")`

1. Modify the code as follows:

    ```Python
   customers = df.select("CustomerName", "Email").where(df['Item']=='Road-250 Red, 52')
   print(customers.count())
   print(customers.distinct().count())
   display(customers.distinct())
    ```

1. Run the modified code to view the customers who have purchased the *Road-250 Red, 52* product. Note that you can "chain" multiple functions together so that the output of one function becomes the input for the next - in this case, the dataframe created by the **select** method is the source dataframe for the **where** method that is used to apply filtering criteria.

1. Add a new code cell to the notebook, and enter the following code in it:

    ```Python
   productSales = df.select("Item", "Quantity").groupBy("Item").sum()
   display(productSales)
    ```

1. Run the code cell you added, and note that the results show the sum of order quantities grouped by product. The **groupBy** method groups the rows by *Item*, and the subsequent **sum** aggregate function is applied to all of the remaining numeric columns (in this case, *Quantity*)

1. Add another new code cell to the notebook, and enter the following code in it:

    ```Python
   from pyspark.sql.functions import *

   yearlySales = df.select(year(col("OrderDate")).alias("Year")).groupBy("Year").count().orderBy("Year")
   display(yearlySales)
    ```

1. Run the code cell you added, and note that the results show the number of sales orders per year. Note that the **select** method includes a SQL **year** function to extract the year component of the *OrderDate* field (which is why the code includes an **import** statement to import functions from the Spark SQL library). It then uses an **alias** method is used to assign a column name to the extracted year value. The data is then grouped by the derived *Year* column and the count of rows in each group is calculated before finally the **orderBy** method is used to sort the resulting dataframe.

## Task 4 : Use Spark to transform data files

A common task for data engineers is to ingest data in a particular format or structure, and transform it for further downstream processing or analysis.

1. Add another new code cell to the notebook, and enter the following code in it:

    ```Python
   from pyspark.sql.functions import *

   ## Create Year and Month columns
   transformed_df = df.withColumn("Year", year(col("OrderDate"))).withColumn("Month", month(col("OrderDate")))

   # Create the new FirstName and LastName fields
   transformed_df = transformed_df.withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)).withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

   # Filter and reorder columns
   transformed_df = transformed_df["SalesOrderNumber", "SalesOrderLineNumber", "OrderDate", "Year", "Month", "FirstName", "LastName", "Email", "Item", "Quantity", "UnitPrice", 
   "Tax"]

   # Display the first five orders
   display(transformed_df.limit(5))
    ```

2. Run the code to create a new dataframe from the original order data with the following transformations:
    - Add **Year** and **Month** columns based on the **OrderDate** column.
    - Add **FirstName** and **LastName** columns based on the **CustomerName** column.
    - Filter and reorder the columns, removing the **CustomerName** column.

3. Review the output and verify that the transformations have been made to the data.

    You can use the full power of the Spark SQL library to transform the data by filtering rows, deriving, removing, renaming columns, and applying any other required data modifications.

    > **Tip**: See the [Spark dataframe documentation](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html) to learn more about the methods of the Dataframe object.

4. Add a new cell with the following code to save the transformed dataframe in Parquet format (Overwriting the data if it already exists):

    ```python
   transformed_df.write.mode("overwrite").parquet('Files/transformed_data/orders')
   print ("Transformed data saved!")
    ```

    > **Note**: Commonly, *Parquet* format is preferred for data files that you will use for further analysis or ingestion into an analytical store. Parquet is a very efficient format that is supported by most large scale data analytics systems. In fact, sometimes your data transformation requirement may simply be to convert data from another format (such as CSV) to Parquet!

5. Run the cell and wait for the message that the data has been saved. Then, in the **Explorer** pane on the left, In the menu select **ellipse** icon for the **Files** node, select **Refresh**; and select the **transformed_orders** folder to verify that it contains a new folder named **orders**, which in turn contains one or more Parquet files.

   ![Screenshot of a folder containing parquet files.](./Images/saved-parquet1.png)

6. Add a new cell with the following code to load a new dataframe from the parquet files in the **transformed_orders/orders** folder:

    ```python
   orders_df = spark.read.format("parquet").load("Files/transformed_data/orders")
   display(orders_df)
    ```

7. Run the cell and verify that the results show the order data that has been loaded from the parquet files.

8. Add a new cell with the following code; which saves the dataframe, partitioning the data by **Year** and **Month**:

    ```python
   orders_df.write.partitionBy("Year","Month").mode("overwrite").parquet("Files/partitioned_data")
   print ("Transformed data saved!")
    ```

9. Run the cell and wait for the message that the data has been saved. Then, in the **Explorer** pane on the left, In the menu select **ellipse** icon for the **Files** node, select **Refresh**; and expand the **partitioned_orders** folder to verify that it contains a hierarchy of folders named **Year=*xxxx***, each containing folders named **Month=*xxxx***. Each month folder contains a parquet file with the orders for that month.

    ![Screenshot of a hierarchy of partitioned data files.](./Images/partitioned-files1.png)

    > Partitioning data files is a common way to optimize performance when dealing with large volumes of data. This technique can significantly improve performance and make it 
    easier to filter data.
   

10. Add a new cell with the following code to load a new dataframe from the **orders.parquet** file:

     ```python
    orders_2021_df = spark.read.format("parquet").load("Files/partitioned_data/Year=2021/Month=*")
    display(orders_2021_df)
     ```

11. Run the cell and verify that the results show the order data for sales in 2021. Note that the partitioning columns specified in the path (**Year** and **Month**) are not included in the dataframe.

12. Add a new code cell to the notebook, and enter the following code, which saves the dataframe of sales order data as a table named **salesorders**:

     ```Python
    # Create a new table
    df.write.format("delta").saveAsTable("salesorders")

    # Get the table description
    spark.sql("DESCRIBE EXTENDED salesorders").show(truncate=False)
     ```

    > **Note**: It's worth noting a couple of things about this example. Firstly, no explicit path is provided, so the files for the table will be managed by the metastore. Secondly, the table is saved in **delta** format. You can create tables based on multiple file formats (including CSV, Parquet, Avro, and others) but *delta lake* is a Spark technology that adds relational database capabilities to tables; including support for transactions, row versioning, and other useful features. Creating tables in delta format is preferred for data lakehouses in Fabric.

13. Run the code cell and review the output, which describes the definition of the new table.

14. In the **Explorer** pane, In the menu select **ellipse** icon for the **Tables** folder, select **Refresh**. Then expand the **Tables** node and verify that the **salesorders** table has been created.

    ![Screenshot of the salesorder table in Explorer.](./Images/table-view1.png)

15. In the menu select **ellipse** icon for the **salesorders** table, select **Load data** > **Spark**.

    A new code cell containing code similar to the following example is added to the notebook:

     ```Python
    df = spark.sql("SELECT * FROM [your_lakehouse].salesorders LIMIT 1000")
    display(df)
     ```

16. Run the new code, which uses the Spark SQL library to embed a SQL query against the **salesorder** table in PySpark code and load the results of the query into a dataframe.

17. Add a new code cell to the notebook, and enter the following code in it:

     ```sql
    %%sql
    SELECT YEAR(OrderDate) AS OrderYear,
           SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
    FROM salesorders
    GROUP BY YEAR(OrderDate)
    ORDER BY OrderYear;
     ```

18. Run the cell and review the results. Observe that:
    - The `%%sql` line at the beginning of the cell (called a *magic*) indicates that the Spark SQL language runtime should be used to run the code in this cell instead of PySpark.
    - The SQL code references the **salesorders** table that you created previously.
    - The output from the SQL query is automatically displayed as the result under the cell.

> **Note**: For more information about Spark SQL and dataframes, see the [Spark SQL documentation](https://spark.apache.org/docs/2.2.0/sql-programming-guide.html).

## Task 5 : Visualize data with Spark

A picture is proverbially worth a thousand words, and a chart is often better than a thousand rows of data. While notebooks in Fabric include a built-in chart view for data that is displayed from a dataframe or Spark SQL query, it is not designed for comprehensive charting. However, you can use Python graphics libraries like **matplotlib** and **seaborn** to create charts from data in dataframes.

1. Add a new code cell to the notebook, and enter the following code in it:

    ```sql
   %%sql
   SELECT * FROM salesorders
    ```

2. Run the code and observe that it returns the data from the **salesorders** view you created previously.

3. In the results section beneath the cell, change the **View** option from **Table** to **Chart**.

4. Use the **View options** button at the top right of the chart to display the options pane for the chart. Then set the options as follows and select **Apply**:
    - **Chart type**: Bar chart
    - **Key**: Item
    - **Values**: Quantity
    - **Series Group**: *leave blank*
    - **Aggregation**: Sum
    - **Stacked**: *Unselected*

5. Verify that the chart looks similar to this:

    ![Screenshot of a bar chart of products by total order quantiies](./Images/chart.png)

## Review

In this exercise, data analysis was conducted with Apache Spark SDP -Fabric. The process began with the creation of a notebook to serve as the analytical environment. Data was loaded into a DataFrame for further processing. Exploration of the DataFrame provided insights into the dataset's characteristics. Spark was then employed to transform data files, enhancing their structure and content. Finally, data visualization with Spark facilitated a comprehensive understanding of patterns and trends within the dataset.

## Proceed to next exercise
