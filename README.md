# SQL Server Long-Running Query Monitoring

## Objective

The SQL Server Long Running Query Monitoring project aimed to automatically identify SQL Server queries that run longer than a defined time threshold and notify the database administrator through email. The goal was to improve proactive database performance monitoring by detecting long running queries, blocking sessions, and potentially inefficient SQL statements. This hands on project provided practical experience with SQL Server Dynamic Management Views (DMVs), query performance analysis, execution plans, SQL Server Profiler, indexing, statistics, T-SQL optimization, and automated Database Mail notifications.

### Skills Learned

* Monitoring SQL Server active requests using Dynamic Management Views (DMVs).
* Identifying long running queries and blocking sessions.
* Analyzing SQL statements and query execution performance.
* Using SQL Server Profiler to identify high read operations.
* Analyzing Actual Execution Plans to locate expensive query operations.
* Reviewing and optimizing indexes based on query performance requirements.
* Checking and updating table and index statistics when required.
* Optimizing stored procedures and complex SQL logic.
* Using `SET NOCOUNT ON` in stored procedures.
* Creating HTML formatted email reports using T-SQL.
* Configuring SQL Server Database Mail for automated alerts.
* Applying practical SQL Server performance troubleshooting techniques.

### Tools Used

* **Microsoft SQL Server 2016** for database administration and performance monitoring.
* **SQL Server Management Studio (SSMS)** for T-SQL development, execution plans, and query analysis.
* **T-SQL** for stored procedure development and optimization.
* **SQL Server Dynamic Management Views (DMVs)** for monitoring active sessions and requests.
* **SQL Server Profiler** for analyzing query activity and read operations.
* **Execution Plans** for identifying expensive operators and performance bottlenecks.
* **SQL Server Database Mail** for automated email notifications.

## Steps

Below are the key steps taken in the long running query monitoring process:

### 1. Monitor Active SQL Server Requests

The monitoring procedure uses SQL Server Dynamic Management Views to identify currently executing requests and collect information about active sessions.

The following DMVs and functions were used:

* `sys.dm_exec_requests`
* `sys.dm_exec_sessions`
* `sys.dm_exec_connections`
* `sys.dm_exec_sql_text()`

The procedure captures information such as the session ID, status, login, host, blocking session, command type, elapsed time, start time, and currently executing SQL statement.

*Ref 1: Stored Procedure*
This screenshot shows the SQL Server session and request information used to monitor currently executing queries.

![Stored Procedure](https://github.com/Vishnupd/SQL-Server-Long-Running-Query-Monitoring-/blob/main/SP_1.png)
![Stored Procedure](https://github.com/Vishnupd/SQL-Server-Long-Running-Query-Monitoring-/blob/main/SP_2.png)

### 2. Identify Long-Running Queries

The procedure checks the execution time of active requests and identifies queries that have been running for more than 60 seconds.

```sql
WHERE st.text IS NOT NULL
  AND er.total_elapsed_time / 1000 > 60
```

The `total_elapsed_time` value is converted from milliseconds to seconds before applying the threshold.

*Ref 2.1: Creating a Long Running Query Intentionally with WAITFOR DELAY as an Exmaple*
This screenshot shows intentionally creating long running query.

![Long-Running Query Creation](https://github.com/Vishnupd/SQL-Server-Long-Running-Query-Monitoring-/blob/main/Longrunning_Query_with_WaitForDelay.png)

*Ref 2.2: Long Running Query Detection*
This screenshot shows the query monitoring logic used to identify requests exceeding the 60 second execution threshold.

![Long-Running Query Detection](https://github.com/Vishnupd/SQL-Server-Long-Running-Query-Monitoring-/blob/main/Long%20running%20Query%20_Detected.png)

### 3. Capture Query and Blocking Information

Once a long running query is detected, the procedure captures detailed information about the request, including the SPID, login, host, blocking session ID, command type, elapsed time, start time, and SQL statement.

The `blocking_session_id` value helps identify whether another SQL Server session is blocking the detected request.

*Ref 3: Query and Session Details*
This screenshot shows the detailed SQL Server session information captured for the detected long-running query, including blocking information and the SQL statement.

![Query and Session Details](https://github.com/Vishnupd/SQL-Server-Long-Running-Query-Monitoring-/blob/main/Long%20running%20Query%20_Detected.png)

### 4. Generate an HTML Monitoring Report

The procedure converts the captured query information into XML and uses it to construct an HTML table for the email notification.

```sql
FOR XML PATH('tr'), ELEMENTS
```

The generated report contains details such as:

* SPID
* Status
* Login
* Host
* Blocking Session
* Command Type
* Elapsed Time
* Start Time
* Time Elapsed
* SQL Statement

HTML and CSS formatting were added to make the monitoring report easier to read.

### 5. Send Automated Database Mail Notification

When a long-running query is detected, the procedure uses SQL Server Database Mail to automatically send an email notification.

```sql
EXEC msdb.dbo.sp_send_dbmail
    @profile_name = 'SQL Server Mail Profile',
    @body = @body,
    @body_format = 'HTML',
    @recipients = 'vishnuprasad19931994@gmail.com',
    @subject = ' - Long running query detected';
```

The email is sent in HTML format so that the captured query information is displayed as a structured report.

### 6. Receive Long-Running Query Alert

When a query exceeds the 60-second threshold, an automated email alert is generated containing the query and session information.

The notification helps the database administrator quickly identify:

* Long-running queries.
* Blocking sessions.
* SQL statements requiring investigation.
* Login and host information.
* Query execution time.
* Query start time.

*Ref 6: Long-Running Query Alert*
This screenshot shows the automated HTML email received when a long-running query was detected.

![Long-Running Query Alert](https://github.com/Vishnupd/SQL-Server-Long-Running-Query-Monitoring-/blob/main/Longrunning_Query_Email.png)

### 7. Investigate and Troubleshoot Long-Running Queries

The captured information was reviewed in detail to identify the root cause of the long-running query. The SQL statement was analyzed for inefficient logic, complex joins, unnecessary operations, and other potential performance issues.

Basic optimization techniques were applied where necessary, including reviewing the query logic, simplifying complex operations, and checking whether the query could be improved.

SQL Server Profiler was used to analyze query activity and identify queries performing high numbers of reads.

The Actual Execution Plan was reviewed to locate expensive operators, table scans, index scans, joins, and other areas contributing to the query cost.

Existing indexes were checked, and indexes were added or modified only when required and supported by the query workload and execution plan.

Table and index statistics were also reviewed. Statistics were updated when necessary, and indexes were rebuilt or reorganized when fragmentation was identified as a contributing factor.

Stored procedure logic was reviewed as part of the optimization process. `SET NOCOUNT ON` was used where appropriate, and the code logic was modified when complex joins or inefficient processing were identified.

After making the required changes, the query was re-tested to compare execution time, reads, and overall performance.

