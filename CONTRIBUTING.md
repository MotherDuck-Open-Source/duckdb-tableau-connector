# Developing DuckDB Tableau Connector

## Notes on manifest.xml
### <connector-plugin> tag
* `plugin-version` attribute is the version seen by users in Tableau Exchange. 
Increment it whenever the connector changes.
* `version` attribute refers to an internal Tableau version and should not be changed.

### <connection-customization>
* `version` attribute is not used by Tableau.
Leave it at 1.0 unless connection-customization changes.

## Releasing

# Update user-agent connector version
In `connectionProperties.js`, please update `MOTHERDUCK_CONNECTOR_VERSION` to the latest version.