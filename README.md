##README.md

Hive schema:

create external table nyc_trips_pq 
(
vendor_name         string,                                  
trip_pickup_datetime string,                                  
trip_dropoff_datetime string,                                  
passenger_count     int,                                  
trip_distance       float,                                  
payment_type        string,                                  
are_amt             float,                                   
surcharge           float,                                   
mta_tax             float,                                   
tip_amt             float,                                   
tolls_amt           float,                                   
total_amt           float
)
PARTITIONED BY (year String, month String)  
STORED AS PARQUET
LOCATION 's3://neilawspublic/dataset2/'
tblproperties ("parquet.compress"="SNAPPY");

msck repair table nyc_trips_pq;

Zeppelin Code:

%pyspark
from pyspark.sql import SparkSession
from  pyspark.sql import SQLContext

hivetablename='nyc_trips_pq'
sqltext='Select year, month,sum(Passenger_Count) as total_passengers, count(1) as total_trips from nyc_trips_pq group by year, month order by 4 DESC LIMIT 10'

%pyspark

spark = SparkSession.builder.appName("Zeppelin-Spark").enableHiveSupport().getOrCreate()
df=spark.sql(sqltext)
df.createOrReplaceTempView("nyc_trips")
sql=SQLContext(spark)
sql.cacheTable("nyc_trips")
df.show()


%sql
Select * from nyc_trips

%sql
Select * from nyc_trips


R-Studio:

Script Location:s3://aws-bigdata-blog/artifacts/aws-blog-emr-rstudio-sparklyr/rstudio_sparklyr_emr5.sh

Script Arguments:
--rstudio --shiny --sparkr --rexamples --plyrmr --rhdfs --sparklyr


>> library(sparklyr)
>> sc <- spark_connect(master = "yarn-client", version = "2.1.0")
library(DBI)
>> nyc_trips_preview <- dbGetQuery(sc, "Select year, month,sum(Passenger_Count) as total_passengers, count(1) as total_trips from nyc_trips_pq group by year, month order by 4 DESC LIMIT 10")
>> library(ggplot2)
>> bp<- ggplot(nyc_trips_preview, aes(x="", y=total_passengers, fill=year))+
geom_bar(width = 1, stat = "identity")
>> pie <- bp + coord_polar("y", start=0)
>> require(scales)
>> pie + scale_y_continuous(labels = comma)

Jupyter:

Script Location:s3://aws-bigdata-blog/artifacts/aws-blog-emr-jupyter/install-jupyter-emr5.sh

Script Arguments: --r --julia --toree --torch --ruby --ds-packages --ml-packages --python-packages 'ggplot nilearn' --port 8880 --password jupyter --jupyterhub --jupyterhub-port 8001 --cached-install --notebook-dir,s3://neilawstemp/notebooks/ --copy-samples

Jupyter Pyspark Code:

from pyspark.sql import SparkSession
from  pyspark.sql import SQLContext

hivetablename='nyc_trips_pq'
sqltext='Select year, month,sum(Passenger_Count) as total_passengers, count(1) as total_trips from nyc_trips_pq group by year, month order by 4 DESC LIMIT 10'

spark = SparkSession.builder.appName("Zeppelin-Spark").enableHiveSupport().getOrCreate()

df=spark.sql(sqltext)
df.createOrReplaceTempView("nyc_trips")
sql=SQLContext(spark)
sql.cacheTable("nyc_trips")
df.show()



