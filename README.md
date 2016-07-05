sqlcc
=====

Watch this for a quick summary: http://screencast.com/t/5I2N4GQgcb4 (please note: this video's syntax is out of date.  Please see below for updated syntax).

**The Skinny**

SQLCC is a code coverage tool that allows developers the ability to track which lines of code within stored procedures, functions, and triggers have been tested in SQL server by integration tests written by the developer.

**The Longer Skinny**

We all know it's just silly to stuff business logic in our database-- its harder to maintain, database servers don't scale well, etc.  We could go on and on with reasons why you shouldn't put business logic in SQL.  However, there are times when you inherit that really ancient code-base.  You know, the one with 80-90% of the business rules written in SQL-- yes, that bad boy.  How in Poseidon's Trident do you refactor without breaking something?

The answer?  Meet SQLCC.  It's simple-- just do what you would normally do when faced with legacy code: write some tests and then get refactoring.  To get started, write some integration tests that call a stored procedure, function, or trigger and then use SQLCC to detect the code-coverage for your tests in SQL prior to your refactor to ensure you have covered all possible code-paths.  Then, start refactoring and change that good old integration test to a unit test while moving the business logic into your application code.

For now, Microsoft SQL Server is supported, but has the capability of being expanded to support additional database servers.

## The Codez

### Configuration

Open `SQLCC/App.config` and modify the parameters. The important ones are:
- `databaseConnectionString`, which is your connection string to the database which you are running your tests against
- `databaseApplicationName` is your Application Name as described below under "How Does it Work?"
- `databaseTraceDirectory` is the directory which you would like to store your trace files (currently outputs \*.trc files on the server)-- this is on the SQL server **(must have read/write access to this directory)**
- `outputDirectory` is the output directory for the HTML files that are generated.

### Running tests from SQLCC command

In its most simple form, to run MSTest tests and execute SQLC, just run the following:

    sqlcc --action=execute --target="mstest.exe" --targetArgs="/testcontainer:SQLCC.Sample.Tests.dll" --traceFileName="TraceFile"
    sqlcc --action=generate --traceFileName="TraceFile"

The first line, where action is set to `execute`, starts a MSSQL trace with the appropriate settings enabled in order to track which lines have been executed.

The second line generates a set of HTML files with the code coverage from the specified Trace. You can also create your own OutputProvider and store the results in another file format or in the database for querying.

### Running tests separately

If you want to run your tests separately from SQLCC, you can run the following commands:

- `sqlcc --action=start --traceFileName="TraceFile"` (creates and starts a MSSQL Trace)
- Run your tests
- `sqlcc --action=stop --traceFileName="TraceFile"` (stops the MSSQL Trace)
- `sqlcc --action=generate --traceFileName="TraceFile"` (generates HTML reports)

In the project root, you can find three .bat scripts for each of these actions (`start`, `stop` and `generate`). You can configure them to make things easier.

## How Does it Work?

It works by simply running a trace with a few settings enabled in order to detect code execution paths.  In order to limit traffic to just your running tests, one would simply pass in an Application Name to sqlcc and set up your tests to run using a connection string like the following:

    Data Source=localhost;Initial Catalog=MY_DB;User Id=sa;Password=password;Application Name=SQLCC;

Notice "SQLCC" as the Application Name.
 
## Is this done?

Yes, for the most part, its a really rough proof of concept (aka alpha). Definitely needs some love and unfortunately I'm not able to fully devote myself to this project.

## Troubleshooting

- **You do not have permission to run SYS.TRACES**: you need to grant alter trace permissions to your user. This command fixes that (run it on `master` db): `GRANT ALTER TRACE TO [youruser]`

- **System.Data.SqlClient.SqlException: Unable to create trace file**: probably the MSSQL user doesn't have read/write access to the trace directory. You can manually set these permissions to the `NT SERVICE\MSSQLSERVER` user **or** run [this PowerShell script](https://gist.github.com/stefanteixeira/914b71422caa77117d99) (configuring your directory correctly in the script)

- Ensure that "target" is pointing to the full path of your test runner.  If this is MSTest, then include the full path to "mstest.exe", or if it is NUnit, include the full path to "nunit.exe".

- Ensure you have the correct "targetArgs" for your test runner.  If you are using MSTest and your target is set to "C:\Path\To\mstest.exe", then try running your test runner (mstest.exe, nunit.exe, etc.) without sqlcc first with the appropriate arguments.  Once you can run it correctly, copy over the test runner's arguments to "targetArgs".
