# AWS-Incremental-Load

Incremental data load using AWS Glue 

Tech used->AWS IAM,AWS Glue,Amazon s3 

descp->Designed a Event-driven data pipeline used to compare the incoming data from the source system with the existing data present in the destination with inbuilt glue capabilities.


Used for the common use case every day getting the data in a specific location in S3 and need to be ingested in the same table.
Created a s3 bucket and added the source file which will incrementally add the file to glue crawlers with a database in a table.
Applied inbuilt capabilities such as job bookmark need to be enable for incremental data load and newly arrived file will get processed.
Defined transformation_ctx in glue script used for internal checkpointing in incremantal loading.




Glue Crawlers-
step1:set crawlers properties -create crawlers

2.Choose the data source and classifier
->data source path in S3 
  and set the subsequent folder run
added classifieer (optional csv)

3.step3:Configure security settings:
       create new IAM role ->attach policy
          aws glue serive role->s3/.. access

step4:set output(store the metadata) and scheduling
      .Target database:
             add->batch2-db
                 craete and attach
      .table name prefix->employee_
   Crawler schedule->dynamic changes in file 

4.Create a new GLue script for INcremental data load from glue catelog



name->Salesorder_Data_Processing
1.script
2.Job details->after set the Job deatils:
 job bookmark->enable

in the incremental data load only the newly arrived file will get processed not the previous one.
3.run->
4.scheduler->
5.version control->

to prevent this checkpoint ->it shoud know that what it processed previously with the help of job bookmarking
in job details -> enable (do the internal check pointing)

2.transformation context->craete/write using glue context
used as check point for this tnx ctx it has processed data

salesorderDF=glueContext.create_dynamic_frame.from_catalog(
           database='batch2-database',
           table_name='salesorder_batch2_data_s3'
           transformation_ctx='s3_input_new'
)



3.after spark = glueContext.spark_session
initialize

4.job.commit()

job = Job(glueContext)
job.init(args['JON_NAME'].args)


script:

import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

## @params: [JOB_NAME]
args = getResolvedOptions(sys.argv, ['JOB_NAME'])

sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session

job = Job(glueContext)
job.init(args['JOB_NAME'], args)

salesorderDF=glueContext.create_dynamic_frame.from_catalog(
           database='batch2-database',
           table_name='salesorder_batch2_data_s3',
           transformation_ctx='s3_input_new'
           )


salesorderDF.printSchema()
SaprkSalesDf=salesorderDF.toDF()
SaprkSalesDf.show()
print(SaprkSalesDf.count())



job.commit()  

save->

run and check logs
incremetally proces and load the data
