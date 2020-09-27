# Project: Data Pipelines with Airflow
A music streaming company, Sparkify, has decided that it is time to introduce more automation and monitoring to their data warehouse ETL pipelines and come to the conclusion that the best tool to achieve this is Apache Airflow.

## Datasets
Here are the s3 links for datasets used in this project:

`Log data: s3://udacity-dend/log_data`

`Song data: s3://udacity-dend/song_data`

## Structure

Project has two directories named `dags` and `plugins`. A create tables script and readme file are at root level:


`dags` directory contains:
- `sparkify_etl_dag.py`: Defines main DAG, tasks and link the tasks in required order.
- `sparkify_sub.py`: Defines subDAG that load Dimension Tables.

`plugins/operators` directory contains:
- `stage_redshift.py`: Defines `StageToRedshiftOperator` to copy JSON data from S3 to staging tables in the Redshift via `copy` command.
- `load_dimension.py`: Defines `LoadDimensionOperator` to load a dimension table from staging table(s).
- `load_fact.py`: Defines `LoadFactOperator` to load fact table from staging table(s).
- `data_quality.py`: Defines `DataQualityOperator` to run data quality checks on all tables passed as parameter.

`plugins/helpers` directory contains:
- `sql_queries.py`: Contains SQL queries for the ETL pipeline (provided in template).
- `create_tables.sql`: SQL create table statements provided with template.

### Operators

1. Stage operator 
    * Loads JSON and CSV files from S3 to Amazon Redshift
    * Creates and runs a `SQL COPY` statement based on the parameters provided
    * Parameters should specify where in S3 file resides and the target table
    * Parameters should distinguish between JSON and CSV files
    * Contain a templated field that allows it to load timestamped files from S3 based on the execution time and run backfills
2. Fact and Dimension Operators
    * Use SQL helper class to run data transformations
    * Take as input a SQL statement and target database to run query against
    * Define a target table that will contain results of the transformation
    * Dimension loads are often done with truncate-insert pattern where target table is emptied before the load
    * Fact tables are usually so massive that they should only allow append type functionality
3. Data Quality Operator
    * Run checks on the data
    * Receives one or more SQL based test cases along with the expected results and executes the tests
    * Test result and expected results are checked and if there is no match, operator should raise an exception and the task should retry and fail eventually
    
## Config

This code uses `python 3` and assumes that Apache Airflow is installed and configured.

- Create a Redshift cluster and run `create_tables.sql` there for once only.
- Make sure to add following two Airflow connections:
    - AWS credentials, named `aws_credentials`
    - Connection to Redshift, named `redshift`

### Add Airflow Connections
Here, we'll use Airflow's UI to configure your AWS credentials and connection to Redshift.

To go to the Airflow UI:
You can use the Project Workspace here and click on the blue Access Airflow button in the bottom right.
If you'd prefer to run Airflow locally, open http://localhost:8080 in Google Chrome (other browsers occasionally have issues rendering the Airflow UI).
Click on the Admin tab and select Connections.


![Admin Tab](https://github.com/raefezzeldin/pipelinewithairflow/blob/master/Screenshots/admin-connections.png)


Under Connections, select Create.

![Admin Tab](https://github.com/raefezzeldin/pipelinewithairflow/blob/master/Screenshots/create-connection.png)

On the create connection page, enter the following values:


![Admin Tab](https://github.com/raefezzeldin/pipelinewithairflow/blob/master/Screenshots/connection-aws-credentials.png)


1. Conn Id: Enter aws_credentials.
2. Conn Type: Enter Amazon Web Services.
3. Login: Enter your Access key ID from the IAM User credentials you downloaded earlier.
4. Password: Enter your Secret access key from the IAM User credentials you downloaded earlier.

Once you've entered these values, select Save and Add Another.


On the next create connection page, enter the following values:

![Admin Tab](https://github.com/raefezzeldin/pipelinewithairflow/blob/master/Screenshots/connection-redshift.png)

1. Conn Id: Enter redshift.
2. Conn Type: Enter Postgres.
3. Host: Enter the endpoint of your Redshift cluster, excluding the port at the end. You can find this by selecting your cluster in the Clusters page of the Amazon Redshift console. See where this is located in the screenshot below. IMPORTANT: Make sure to NOT include the port at the end of the Redshift endpoint string.
4. Schema: Enter dev. This is the Redshift database you want to connect to.
5. Login: Enter awsuser.
6. Password: Enter the password you created when launching your Redshift cluster.
7. Port: Enter 5439.

Once you've entered these values, select Save.
