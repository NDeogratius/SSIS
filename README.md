# ETL - SQL Server Integration Services
## SSIS - Power Query Source 

1. **Launch Power BI and Connect to Data Source**: Open Power BI and connect to your data source. Once connected, click on `Transform Data` to launch Power Query.

2. **Perform Necessary Transformations**: Use Power Query to perform the necessary transformations on your data. 

3. **Reveal the M-Query**: After you've completed your transformations, go to the `View` ribbon and click on `Advanced Editor`. This will reveal the M-query that's behind the transformations you've carried out.

4. **Copy the M-Query**: Select the code in the `Advanced Editor` window and copy it.

5. **Create a New SSIS Project in Visual Studio**: Launch Visual Studio and create a new SSIS project. 

6. **Add a Data Flow Control to Control Flow**: In your SSIS project, add a `Data Flow Control` to the `Control Flow`.

7. **Add a Power Query Source Task**: Edit the `Data Flow Task` and add a `Power Query Source` task.

8. **Paste the M-Query**: In the `Power Query Source` task, select `Single Query` in the `Query Mode` and paste the M-Query you copied from Power BI into the `Single Query` text box.
![alt text](https://github.com/NDeogratius/SSIS/blob/main/Power%20Query%20Source/powerquerysourceeditor.png)
9. **Configure Connection Managers**: Go to `Connection Managers`, click `Add` to add a new connection manager.
![alt text](https://github.com/NDeogratius/SSIS/blob/main/Power%20Query%20Source/powerquerysourceconmgr.png)
11. **Choose a Data Source Kind**: In the `Connection Manager Editor`, choose a `Data Source Kind`. In this case, select `File` and paste the link to the file in the `Data Source Path`. Also, select the appropriate `Authentication Kind`.
![alt text](https://github.com/NDeogratius/SSIS/blob/main/Power%20Query%20Source/powerquerysourceconmgreditor.png)
13. **Test the Connection**: Click on `Test Connection` to ensure the connection is successful. If the test is successful, you've successfully configured the SSIS Power Query source.

## Note

[In SQL Server Integration Services (SSIS), when you use Power Query as a source, text columns might be displayed as `Long Text`. This is because the source outputs the text columns as Unicode text stream (DT_NTEXT), which cannot be displayed in the data viewer.](https://www.mssqltips.com/sqlservertip/6333/sql-server-integration-services-power-query-source/)

[The DT_TEXT and DT_NTEXT are text data types in SSIS and there are several limitations related to handling these data types. For example, none of the string functions would work with these data types.](https://www.mssqltips.com/sqlservertip/6333/sql-server-integration-services-power-query-source/)

[Power Query is a technology that allows you to connect to various data sources and transform data using Excel/Power BI Desktop3. The script generated by Power Query can be copied & pasted into the Power Query Source in the SSIS data flow to configure it.](https://www.mssqltips.com/sqlservertip/6333/sql-server-integration-services-power-query-source/)

[Please note that not every possible action or source from Power BI Desktop might be supported, or the component itself might still have some bugs.](https://www.mssqltips.com/sqlservertip/6333/sql-server-integration-services-power-query-source/) So, it’s important to keep these limitations in mind when working with Power Query in SSIS. If you need to perform operations on these text columns, you might need to convert them to a different data type that supports the operations you need.

Remember to save your work regularly and always double-check your configurations to avoid errors. Happy data transforming!

---
## SSIS - Dynamic Column Headers

In the realm of ETL, it's common to receive data daily, with each day's data being appended as a new column. At the end of the month, the column count resets, and the cycle begins anew. As an ETL professional using SSIS, you can efficiently manage this scenario with a script package. The process involves the following steps:

#### Steps to Accomplish the Task

1. **Create Necessary Variables**:
   - **File Path**: The path to the source data file.
   - **Server Name**: The name of the server hosting the database.
   - **Database Name**: The name of the target database.
   - **Table Name**: The name of the table where the data will be stored.

These variables will enable you to dynamically handle the incoming data and automate the ETL process, ensuring smooth daily updates and monthly resets.
This script is a C# code snippet intended for use in an SSIS (SQL Server Integration Services) package. It reads data from an Excel file and imports it into a specified SQL Server database table. Below is a detailed explanation of the script:

2. **Add a Script Task**
This task is added to the control flow and not the data flow. Add the above variables as read only variables to the script task editor as show below
![alt text](https://github.com/NDeogratius/SSIS/blob/main/Dynamic%20Column%20Headers/edit-script.png)

4. **Edit the script**
Click on `Edit Script` and add the following `csharp` script

### Main Method

The `Main` method serves as the entry point of the script.

#### Error Logging Setup

1. **Current Date and Time String**:
    ```csharp
    string currentdatetime = DateTime.Now.ToString("yyyyMMddHHmmss");
    ```
    Generates a string with the current date and time in the format `yyyyMMddHHmmss`.

2. **Log Folder Path**:
    ```csharp
    string LogFolder = @Dts.Variables["ScriptErrorLog"].Value.ToString();
    ```
    Retrieves the path for storing error logs from the SSIS package variable `ScriptErrorLog`.

#### Main Processing Logic

3. **Try Block**: Contains the main logic to ensure any exceptions are caught and logged.
    ```csharp
    try
    {
        ...
    }
    ```

4. **Retrieve SSIS Package Variables**:
    ```csharp
    string FilePath = @Dts.Variables["FileName"].Value.ToString();
    string Server = @Dts.Variables["ServerName"].Value.ToString();
    string Database = @Dts.Variables["StageDB"].Value.ToString();
    string TableName = Dts.Variables["TableName"].Value.ToString();
    ```
    These lines fetch values for `FilePath`, `ServerName`, `StageDB`, and `TableName` from SSIS package variables.

5. **Set Up Excel Connection**:
    ```csharp
    string ConStr = "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=" + FilePath + ";Extended Properties=\"Excel 12.0;HDR=YES;IMEX=1\"";
    OleDbConnection cnn = new OleDbConnection(ConStr);
    ```
    Constructs a connection string for the Excel file and initializes an `OleDbConnection`.

6. **Set Up SQL Server Connection**:
    ```csharp
    string sqlconnectionstring = @"Data Source =" + Server + "; Initial Catalog =" + Database + "; persist security info = True; Integrated Security = SSPI;";
    ```
    Constructs a connection string for the SQL Server database.

7. **Open Excel File**:
    ```csharp
    cnn.Open();
    DataTable Sheet = cnn.GetOleDbSchemaTable(OleDbSchemaGuid.Tables, null);
    ```
    Opens the Excel file and retrieves a schema table containing the sheet names.

8. **Iterate Over Sheets**:
    ```csharp
    foreach (DataRow DrSheet in Sheet.Rows)
    {
        if (DrSheet["TABLE_NAME"].ToString().Contains("$"))
        {
            sheetname = DrSheet["TABLE_NAME"].ToString();
            ...
        }
        break;
    }
    ```
    Iterates through the sheets, processes the first sheet that contains data (indicated by `$`).

9. **Read Excel Data**:
    ```csharp
    OleDbCommand oconn = new OleDbCommand("select * from [" + sheetname + "]", cnn);
    OleDbDataAdapter adp = new OleDbDataAdapter(oconn);
    DataTable dt = new DataTable();
    adp.Fill(dt);
    ```
    Executes a query to select all data from the sheet and fills it into a `DataTable`.

10. **Generate SQL DDL for Table Creation**:
    ```csharp
    string tableDDL = "IF EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'[dbo].[" + TableName + "]') AND type in (N'U')) Drop Table [dbo].[" + TableName + "]; Create table [" + TableName + "] (";
    for (int i = 0; i < dt.Columns.Count; i++)
    {
        if (i != dt.Columns.Count - 1)
            tableDDL += "[" + dt.Columns[i].ColumnName + "] " + "NVarchar(255)" + ",";
        else
            tableDDL += "[" + dt.Columns[i].ColumnName + "] " + "NVarchar(255)";
    }
    tableDDL += ")";
    ```
    Constructs a SQL script to drop the existing table and create a new table with columns matching the `DataTable` schema.

11. **Execute SQL Commands**:
    ```csharp
    SqlConnection sqlCon = new SqlConnection(sqlconnectionstring);
    sqlCon.Open();
    SqlCommand command = new SqlCommand(tableDDL, sqlCon);
    command.CommandTimeout = 0;
    command.ExecuteNonQuery();
    ```
    Opens a connection to the SQL Server, creates the table, and executes the DDL.

12. **Bulk Copy Data to SQL Server**:
    ```csharp
    SqlBulkCopy blk = new SqlBulkCopy(sqlCon);
    blk.DestinationTableName = "[" + TableName + "]";
    blk.WriteToServer(dt);
    sqlCon.Close();
    cnn.Close();
    ```
    Uses `SqlBulkCopy` to efficiently transfer the data from the `DataTable` to the SQL Server table.

#### Error Handling

13. **Catch Block**:
    ```csharp
    catch (Exception exception)
    {
        using (StreamWriter sw = File.CreateText(LogFolder + "\\" + "ErrorLog_" + currentdatetime + ".log"))
        {
            sw.WriteLine(exception.ToString());
        }
    }
    ```
    Catches any exceptions that occur during the try block, writes the error details to a log file.

### Final Step

14. **Set Task Result**:
    ```csharp
    Dts.TaskResult = (int)ScriptResults.Success;
    ```
    Marks the SSIS task as successful.

In summary, this script reads data from an Excel file and imports it into a SQL Server table, handling errors by logging them to a file.

**The full script**
 ```csharp
 public void Main()
{
    // TODO: Add your code here

    //variabales to name and store error log file. In case the script encounters and error

    string currentdatetime = DateTime.Now.ToString("yyyyMMddHHmmss");
    string LogFolder = @Dts.Variables["ScriptErrorLog"].Value.ToString(); ;

    try
    {
        //variables used by the script
        string FilePath = @Dts.Variables["FileName"].Value.ToString();
        string Server = @Dts.Variables["ServerName"].Value.ToString();
        string Database = @Dts.Variables["StageDB"].Value.ToString();
        string FileName = Path.GetFileName(FilePath);
        //string TableName = "";
        string TableName = Dts.Variables["TableName"].Value.ToString();
        
        //Connection string to the source file
        string ConStr;
        string HDR;
        HDR = "YES";
        ConStr = "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=" + FilePath + ";Extended Properties=\"Excel 12.0;HDR=" + HDR + ";IMEX=1\"";
        OleDbConnection cnn = new OleDbConnection(ConStr);

        //connection string to database
        string sqlconnectionstring = @"Data Source =" + Server + "; Initial Catalog =" + Database + "; persist security info = True; Integrated Security = SSPI;";

        //Open Excel file
        cnn.Open();
        DataTable Sheet = cnn.GetOleDbSchemaTable(OleDbSchemaGuid.Tables, null);
        string sheetname;
        sheetname = "";
        foreach (DataRow DrSheet in Sheet.Rows)
        {
            if (DrSheet["TABLE_NAME"].ToString().Contains("$"))
            {
                sheetname = DrSheet["TABLE_NAME"].ToString();
                //Get data from the excel file and upload it into a Datatable Object
                OleDbCommand oconn = new OleDbCommand("select * from [" + sheetname + "]", cnn);
                OleDbDataAdapter adp = new OleDbDataAdapter(oconn);
                DataTable dt = new DataTable();
                adp.Fill(dt);

                sheetname = sheetname.Replace("$", "");

                //use the schema of the datatable to create a similar table on the database server
                string tableDDL = "";
                tableDDL += "IF EXISTS (SELECT * FROM sys.objects WHERE object_id = ";
                tableDDL += "OBJECT_ID(N'[dbo].[" + TableName + "]') AND type in (N'U'))";
                tableDDL += "Drop Table [dbo].[" + TableName + "]";
                tableDDL += "Create table [" + TableName + "]";
                tableDDL += "(";
                for (int i = 0; i < dt.Columns.Count; i++)
                {
                    if (i != dt.Columns.Count - 1)
                        tableDDL += "[" + dt.Columns[i].ColumnName + "] " + "NVarchar(255)" + ",";
                    else
                        tableDDL += "[" + dt.Columns[i].ColumnName + "] " + "NVarchar(255)";
                }
                tableDDL += ")";


                SqlConnection sqlCon = new SqlConnection(sqlconnectionstring);
                sqlCon.Open();

                SqlCommand command = new SqlCommand(tableDDL, sqlCon);
                command.CommandTimeout = 0;
                command.ExecuteNonQuery();

                SqlBulkCopy blk = new SqlBulkCopy(sqlCon);
                blk.DestinationTableName = "[" + TableName + "]";
                blk.WriteToServer(dt);
                sqlCon.Close();
                cnn.Close();
            }
            break;
        }
    }
    //get any errors within the script session and write them to the error log
    catch (Exception exception)
    {
        using (StreamWriter sw = File.CreateText(LogFolder + "\\" + "ErrorLog_" + currentdatetime + ".log"))
        {
            sw.WriteLine(exception.ToString());
        }

    }


    Dts.TaskResult = (int)ScriptResults.Success;
}   

 ```


#### Key Points

1. **Automatic Table Creation**:
   The script dynamically creates a table on the SQL Server instance using the column names from the Excel file. This eliminates the need to predefine the column names, making the process more flexible and adaptable to changes in the source file.

2. **Facilitates Staging for Transformations**:
   By ingesting the source file into a staging table, the script allows subsequent data transformations to use this staging table as their input. This staging step ensures that the raw data is readily available for further processing and transformation.

3. **Handling Dynamic Column Headers**:
   Given that each column may contain different dates, indicating that the actual column headers are in the rows of the first column, there is no requirement to know the column names beforehand. The data can be unpivoted in the staging area, allowing for seamless continuation of data transformation and ingestion processes. This approach simplifies handling dynamic and variable column headers.
