# Using an unsigned Taco

You may need to run an unsigned connector, either during development or to test out new features before they get released.

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
