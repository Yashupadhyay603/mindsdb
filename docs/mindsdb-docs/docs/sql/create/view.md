# `#!sql CREATE VIEW` Statement

## Description

The `#!sql CREATE VIEW` statement creates a view, which is a saved `SELECT` statement executed every time we call this view, as opposed to a table that stores its data in columns and rows.

In MindsDB, the `#!sql CREATE VIEW` statement is commonly used to create **AI Tables**. An AI Table is a virtual table created by joining the data source table with the prediction model.

## Syntax

Here is the syntax:

```sql
CREATE VIEW mindsdb.[ai_table_name] AS (
    SELECT
        a.[column_name1],
        a.[column_name2],
        a.[column_name3],
        p.[model_column] AS model_column
    FROM [integration_name].[table_name] AS a
    JOIN mindsdb.[predictor_name] AS p
);
```

On execution, we get:

```sql
Query OK, 0 rows affected (x.xxx sec)
```

Where:

| Name                                  | Description                                                                              |
| ------------------------------------- | ---------------------------------------------------------------------------------------- |
| `[ai_table_name]`                     | Name of the view or the AI Table.                                                        |
| `[column_name1], [column_name2], ...` | Columns of the data source table that are the input for the model to make predictions.   |
| `[model_column]`                      | Name of the target column to be predicted.                                               |
| `[integration_name].[table_name]`     | Data source table name along with the integration where it resides.                      |
| `[predictor_name]`                    | Name of the model.                                                                       |

## Example

Below is the query that creates and trains the `home_rentals_model` model to predict the `rental_price` value. The inner `SELECT` statement provides all real estate listing data used to train the model.

```sql
CREATE PREDICTOR mindsdb.home_rentals_model
FROM integration
    (SELECT * FROM house_rentals_data) AS rentals
PREDICT rental_price AS price;
```

On execution, we get:

```sql
Query OK, 0 rows affected (x.xxx sec)
```

Now, we can [`#!sql JOIN`](/sql/api/join/) the `home_rentals_data` table with the `home_rentals_model` model to make predictions. By creating a view (using the `#!sql CREATE VIEW` statement) that is based on the `SELECT` statement joining the data and model tables, we create an AI Table.

Here, the `SELECT` statement joins the data source table and the model table. The input data for making predictions consists of the `sqft`, `number_of_bathrooms`, and `location` columns. These are joined with the `rental_price` column that stores predicted values.

```sql
CREATE VIEW mindsdb.home_rentals_predictions AS (
    SELECT
        a.sqft,
        a.number_of_bathrooms,
        a.location,
        p.rental_price AS price
    FROM integration.home_rentals_data AS a
    JOIN mindsdb.home_rentals_model AS p
);
```

On execution, we get:

```sql
Query OK, 0 rows affected (x.xxx sec)
```

!!! tip "Dataset for Training and Dataset for Joining"
    In this example, we used the same dataset (`integration.home_rentals_data`) for training the model (see the `CREATE PREDICTOR` statement above) and for joining with the model to make predictions (see the `CREATE VIEW` statement above). It doesn't happen like that in real-world scenarios.
    Normally, you use the old data to train the model, and then you join the new data with this model to make predictions.

    Consider the `old_data` dataset that stores data from the years 2019-2021 and the `new_data` dataset that stores data from the year 2022.

    We train the model with the `old_data` dataset like this:

    ```sql
    CREATE PREDICTOR mindsdb.data_model
    FROM integration
        (SELECT * FROM old_data) AS dataset
    PREDICT column AS predicted_column;
    ```

    Now, having the `data_model` model trained using the `old_data` dataset, we can join this model with the `new_data` dataset to make predictions like this:

    ```sql
    CREATE VIEW mindsdb.data_predictions AS (
        SELECT
            a.column1,
            a.column2,
            a.column3,
            p.column AS predicted_column
        FROM integration.new_data AS a
        JOIN mindsdb.data_model AS p
    );
    ```