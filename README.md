# ADF_Incremental_File_Load
## Designed and deployed a scalable Incremental Data Ingestion pipeline using Azure Data Factory, enabling efficient processing of only new or updated files from Azure Data Lake Storage. Utilized Azure Synapse Analytics to persist and manage the last modified timestamp metadata, significantly improving performance, reducing data redundancy, and optimizing cloud resource utilization.

## Pipeline Flow
![Pipeline Architecture](1-Master.png)

The Pipeline has the following variables : 
![Pipeline Architecture](Pipeline_var.png)

### Activities : 
1. **Get last_timestamp (LOOKUP ACTIVITY) :** This LookUp Activity will Query the Table  dbo.lastModified  and get the Timestamp of the Last File that was modified.
2. **SET Var-lastmodified (SET VARIABLE ACTIVITY) :**  This will set the Pipeline Variable  last_timetamp  with the output of the LOOKUP ACTIVITY .
3. **Source MetaData (GET METADATA) :** This Activity will fetch the details of the Files in the SOURCE FOLDER where the Files are sent over. From this Folder we need to incrementally load the files onto the Destination Folder
4. **Loop Items (FOREACH) :** The Previous Activity will send over an array of the CHILD ITEMS of the SOURCE FOLDER.
5. **Stored Procedure 1 (STORED PROCEDURE) :** This Activity will allow us to run the below Stored Procedure on **Azure Synapse**. This will update the table with the timestamp of the Latest File.

    Parameters of stored procedure : _NewIngestiontime=max_timestamp ; OldIngestionTime=last_timestamp_
   
   ![Pipeline Architecture](Stored_Procedure.png)


Now within the **FOREACH ACTIVITY (Loop Items)** , the below Activities are there:
![Pipeline Architecture](2-Loop-Items-CHILD_PL.png)
1. **File Metadata (GETMETADATA ACTIVITY) :** This will return the _Last Modified time_ of the _ITEM_ which was provided as an input from the _PARENT ACTIVITY_ **Loop Items**
2. **If Condition1 (IF CONDITION ACTIVITY) :** If the timestamp of the _ITEM_ is greater then value stored in the variable _last_timestamp_ then the **TRUE** condition will be activated and the set of activities within it will run (these will copy the FILE from SOURCE to DESTINATION folder and update the variable _temp_max_timestamp_).
3. **Update max timestamp (SET VARIABLE ACTIVITY):** This will update the variable _max_timestamp_ with the value of _temp_max_timestamp_ .

Now, Within the **If Condition1 (IF CONDITION ACTIVITY)** , the below Activities are there:

![Pipeline Architecture](3-If_Condition_1-CHILD.png)

1. **Copy data1 (COPY DATA ACTIVITY) :** This will copy the file from SOURCE to DESTINATION folder that has newly arrived since the last Pipeline Run.
2. **Set intermediate max(SET VARIABLE ACTIVITY) :** This will update the value of the variable _temp_max_timestamp_ with the last modified timestamp of the File .
