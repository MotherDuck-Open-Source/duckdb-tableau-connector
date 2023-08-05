# duckdb-taco
Tableau Connector for DuckDB

While it is possible to use the Tableau-provided Postgres dialect to communicate with the DuckDB JDBC driver,
the experience is not ideal because there are some significant differences between the dialects.
This connector has been fully tested against the Tableau dialect generator and is more compatible 
than the provided Postgres dialect.

Please file any problems as bugs against the this repository and not against DuckDB.

## Installation

Adding new drivers to Tableau is a bit tricky, but hopefully these directions should help.

### JDBC Driver

This connector uses the 0.8.2+ DuckDB JDBC driver. 
This is partly because the ODBC driver does not seem to work well on MacOS.

At a high level, you need to download a recent version of the driver from 
[the nightly JDBC build](https://github.com/duckdb/duckdb/actions/workflows/Java.yml) and 
[install it in the Tableau Drivers directory](https://tableau.github.io/connector-plugin-sdk/docs/drivers#jdbc-driver-class-isolation).
It is expected that starting with the DuckDB 0.9.0 release, the shipping driver will be sufficient.

The links here are for a recent version of the JDBC driver that is compatible with Tableau.
If you wish to connect to a database file,
you will need to make sure the file was created with a file-compatible version of DuckDB.
Also, check that there is only one version of the driver installed as there are multiple filenames in use.

* MacOS Desktop.
  * [Download](https://github.com/duckdb/duckdb/suites/14840102996/artifacts/845094935) the DuckDB JDBC driver. This will a file called `duckdb_jdbc-osx-universal.jar`.
  * Copy it to the `~/Library/Tableau/Drivers/` folder.
* Windows Desktop.
  * [Download](https://github.com/duckdb/duckdb/suites/14840102996/artifacts/845094936) the DuckDB JDBC driver. This will be a file called `duckdb_jdbc.jar`.
  * Copy it to the `C:\Program Files\Tableau\Drivers` directory. 
* Linux Server.
  * Download the DuckDB JDBC driver appropriate for your architecture, either
    * [AMD64](https://github.com/duckdb/duckdb/suites/14840102996/artifacts/845094934) or
    * [AARCH64](https://github.com/duckdb/duckdb/suites/14840102996/artifacts/845094932).
  * Copy it to the `/opt/tableau/tableau_driver/jdbc`.
* Windows Server.
  * [Download](https://github.com/duckdb/duckdb/suites/14840102996/artifacts/845094936) the DuckDB JDBC driver. This will be a file called `duckdb_jdbc.jar`.
  * Copy it to the `C:\Program Files\Tableau\Drivers` directory. 

### Tableau Taco

Tableau Connector files are called "tacos".
The current DuckDB Taco is in the [packaged connector directory](packaged-connector).

The current version of the Taco is not signed, you will need to launch Tableau with signature validation disabled.
(Despite what the Tableau documentation days, the real security risk is in the JDBC driver code,
not the small amount of JavaScript in the Taco.)

#### Server (Online)

On Linux, copy the Taco file to `/opt/tableau/connectors`.
On Windows, copy the Taco file to `C:\Program Files\Tableau\Connectors`.
Then issue the commands to disable signature validation:

```sh
$ tsm configuration set -k native_api.disable_verify_connector_plugin_signature -v true
$ tsm pending-changes apply
```
The last command will restart the server with the new settings.

#### MacOS Desktop

Copy the Taco file to the `/Users/[MacOS User]/Documents/My Tableau Repository/Connectors` folder.
Then launch Tableau Desktop from the Terminal with the the command line argument to disable signature validation:

```sh
$ /Applications/Tableau\ Desktop\ 2023.2.app/Contents/MacOS/Tableau -DDisableVerifyConnectorPluginSignature=true
```

You can also package this up with AppleScript by using the following script:

```
do shell script "\"/Applications/Tableau Desktop 2023.2.app/Contents/MacOS/Tableau\" -DDisableVerifyConnectorPluginSignature=true"
quit
```

Create this file with [the Script Editor](https://support.apple.com/guide/script-editor/welcome/mac) 
(located in `/Applications/Utilities`) and save it as a packaged application. 
You can then double-click it to launch Tableau. 
You will need to change the application name in the script when you get upgrades.

#### Windows Desktop

Copy the Taco file to the `C:\Users\[Windows User]\Documents\My Tableau Repository\Connectors` directory.
Then launch Tableau Desktop from a shell with the the `-DDisableVerifyConnectorPluginSignature=true` argument 
to disable signature validation.

## Connecting

Once the Taco is installed and you have launched Tableau,
you can create a new connection by choosing "DuckDB JDBC by DuckDB Labs":

<img width="1364" alt="Development Mode Connect" src="./images/taco-dev.png">

DuckDB is a file-based database, so the connection dialogue simply asks for a file:

<img width="548" alt="Connection Dialogue" src="./images/taco-connect.png">

Because the engine is embedded in the driver itself, 
you need to make sure that the JDBC driver uses the same database version as was used to create the database file.

Once connected, you can use the Tableau connection window to choose schemas, join tables, 
and perform all the basic data cleaning operations it provides for creating a data source:

<img width="1364" alt="Datasource Editing" src="./images/taco-datasource.png">

Note that DuckDB has a three-level naming system (`database.schema.table`)
so you will need to select the database for the file (instead of `system` or `temp`).

## Initial SQL

Tableau allows connections to run initial SQL.
This feature can be used to make sure that various settings are correct:

<img alt="Initial SQL Example" src="./images/taco-initial-sql.png">

## In-Memory Operation

The driver can be used with an in-memory database by typing the file name `:memory:` 
into the file path.
The data will then need to be provided by an Initial SQL string e.g.,

```sql
CREATE VIEW my_parquet AS
    SELECT *
    FROM read_parquet('/path/to/file/my_file.parquet');
```

You can then access it by using the Tableau Data Source editing controls.

## Use Cases

One of the most powerful uses of DuckDB files is not to access data stored in the file, but to query data in other file formats.
To do this, use the database file to store views of the external files.
For example, you can create a `VIEW` of a directory of Hive-partitioned parquet files.
Or, you can use it to perform fast analytics on a SQLite file.
