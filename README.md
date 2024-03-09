# duckdb-taco - A Tableau Connector for DuckDB

While it is possible to use the Tableau-provided Postgres dialect to communicate with the DuckDB JDBC driver,
the experience is not ideal because there are some significant differences between the dialects.
This connector has been fully tested against the Tableau dialect generator and is more compatible 
than the provided Postgres dialect.

Please file any connectivity problems as bugs against this repository and not against DuckDB.

## Installation

Adding new drivers to Tableau is a bit tricky, but hopefully these directions should help.

### JDBC Driver

The connector works with DuckDB JDBC driver v0.9.2 or higher. 
This is partly because the ODBC driver does not seem to work well on MacOS.

At a high level, you need to download a recent version of the driver and 
[install it in the Tableau Drivers directory](https://tableau.github.io/connector-plugin-sdk/docs/drivers#jdbc-driver-class-isolation).
It is expected that starting with the DuckDB 0.9.0 release, the shipping driver will be sufficient.

The links here are for a recent snapshot of the JDBC driver that is compatible with Tableau.
If you wish to connect to a database file,
you will need to make sure the file was created with a file-compatible version of DuckDB.
Also, check that there is only one version of the driver installed as there are multiple filenames in use.

Download the [snapshot jar](https://oss.sonatype.org/service/local/repositories/snapshots/content/org/duckdb/duckdb_jdbc/0.9.0-SNAPSHOT/duckdb_jdbc-0.9.0-20230806.020824-235.jar)

* MacOS: Copy it to `~/Library/Tableau/Drivers/`
* Windows: Copy it to `C:\Program Files\Tableau\Drivers`
* Linux: Copy it to `/opt/tableau/tableau_driver/jdbc`.

### Tableau Taco

Tableau Connector files are called "tacos".
The DuckDB Taco can be [downloaded here](https://github.com/hawkfish/duckdb-taco/raw/main/packaged-connector/duckdb_jdbc.taco).
The current version of the Taco is not signed, so you will need to launch Tableau with signature validation disabled.
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
(located in `/Applications/Utilities`) 
and [save it as a packaged application](https://support.apple.com/guide/script-editor/save-a-script-as-an-app-scpedt1072/mac):

<img src='/images/taco-applescript.png' alt='tableau-applescript' width=50%>

#### Windows Desktop

Copy the Taco file to the `C:\Users\[Windows User]\Documents\My Tableau Repository\Connectors` directory.
Then launch Tableau Desktop from a shell with the the `-DDisableVerifyConnectorPluginSignature=true` argument 
to disable signature validation.

## Connecting

Once the Taco is installed and you have launched Tableau,
you can create a new connection by choosing "DuckDB JDBC by MotherDuck":

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

<img alt="Initial SQL Example" src="./images/taco-initial-sql.png" width=75%>

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
