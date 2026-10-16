# Setup
## Get the files
You can get all the files mentionned below from:
- [ARCAD-Elias release on GitHub](https://github.com/ARCAD-Software/elias-vscode/releases/latest)
- [ARCAD's Customer Portal](https://portal.arcadsoftware.com/) (Go to the `Products` section and open the `ARCAD Skipper` section.

## Elias REST API Server
The Elias REST API Server comes packaged as a `.war` file. It can run on any operating system that supports Java. It must be deployed on an application server (e.g. IBM i integrated Web Application Server, Jetty, Tomcat...whatever floats your boat!), under the `/elias` context.
A pre-packaged, ready to use Jetty installation is available too. It contains a Jetty runtime packaged with `elias.war`.

### Pre-packaged JETTY IBM i installation
On IBM i, the Elias REST API Server runs as a batch job in its own subsystem, created by the installer. It connects to the IBM i it runs on, so no IBM i address has to be configured.

**Prerequisites for Elias Webservices 26.2 and later (Java 17, Jetty 12)**
- IBM i 7.4 or later.
- Java 17 installed: licensed program 5770-JV1, IBM Semeru Runtime 17. Check that the folder `/QOpenSys/QIBM/ProdData/JavaVM/jdk17/64bit` exists. Java is not bundled in the IBM i installer.
- The user running the installer needs `*ALLOBJ` and `*JOBCTL` special authorities, plus `*SECADM` if the Jetty user profile (`JETTY` by default) does not exist yet.

**Installation**
1. Copy the `Setup_Elias-Webservices-X.Y.Z_IBMi.jar` on the IFS, in the `/tmp` folder.
2. Open a QShell session or connect using SSH
3. Go to the `/tmp` folder and start the installation process
```bash
cd /tmp
java -jar Setup_Elias-Webservices-X.Y.Z-SNAPSHOT_IBMi.jar
```
4. The installation process will ask for several installation values, with default values between brackets. Press ENTER to accept the default values or enter the desired value then press ENTER.

| Question | Default | Notes |
|---|---|---|
| Installation directory | `/home/jetty` | IFS folder for Jetty and `elias.war`. |
| Installation library | `JETTY` | Holds the start/stop commands, job description, job queue and a subsystem, all named after the library. |
| Installation iASP | `*SYSBAS` | |
| HTTP port (0 = disabled) | `8080` | The port your clients (VS Code, CI) connect to. |
| HTTPS port (0 = disabled) | | Enter `0` unless you already have a keystore. See [Enabling HTTPS](#enabling-https-26-2-and-later). |
| Jetty user | `JETTY` | The profile the server job runs under. Created with `*ALLOBJ *JOBCTL` if it does not exist. |


1. After the installation, start the JETTY subsystem to start JETTY web application server. For example, if you installed JETTY in the default `JETTY` library: 
    - `STRSBS JETTY/JETTY`
2. Alternatively, JETTY can started and stopped using the following commands:
    - `JETTY/STRJTYSVR`
    - `JETTY/ENDJTYSVR`

> **Elias Webservices 26.2 and later (Java 17, Jetty 12)**
> HTTPS is enabled by default in the Jetty 12 configuration, but no keystore is shipped. The server does not start until `<jetty_base>/start.d/https.ini` is renamed to `https.ini.off`. On IBM i, answering `0` to the HTTPS port question does this for you. To use HTTPS, provide a keystore and set the properties listed in [Enabling HTTPS](#enabling-https-26-2-and-later) below.

### IBM i folder layout (26.2 and later)
| Path | Content |
|---|---|
| `<installation directory>` | Jetty runtime (`start.jar`, `lib`, `modules`, `etc`) |
| `<installation directory>/base/start.d` | Configuration: `http.ini` (port), `https.ini` |
| `<installation directory>/base/webapps/elias.war` | Elias REST API Server |
| `<installation directory>/base/resources/log4j2.xml` | Logging settings |
| `<installation directory>/base/logs` | Logs |

Java 8 based versions (2.6.x) do not have a `base` folder: everything above is directly under the installation directory.

### Upgrading an IBM i server from a Java 8 version (2.6.x) to 26.2
**Do not install 26.2 over a Java 8 installation.** The folder layout changed (Jetty 12 uses a `base` folder), and the installer does not move your old configuration to it. Install 26.2 **next to** the old server, in a new library and a new directory, then remove the old server once the new one works. You can go back at any time by restarting the old subsystem.

Your data is not affected. The Elias REST API Server is a middleware: custom actions, projects and everything else it works with are stored in the ARCAD IFS folders, not in the Jetty folders. After the upgrade, users have to sign in again.

```
Install Java 17 -> Note old settings -> Stop old server -> Install 26.2 in new library/directory -> Verify -> Update QSTRUP -> Remove old server
```

1. **Check the prerequisites** listed in [Pre-packaged JETTY IBM i installation](#pre-packaged-jetty-ibm-i-installation), especially Java 17.
2. **Note the old settings.** `cat /home/jetty/quadra.installation.properties` shows the answers given to the old installer (directory, library, user...). Also note:
    - the HTTP port: `jetty.http.port` in `/home/jetty/start.d/http.ini`,
    - if you use HTTPS: the `jetty.sslContext.*` lines in `/home/jetty/start.d/https.ini` and the keystore file they point to,
    - your changes to `/home/jetty/resources/log4j2.xml`, if any.
3. **Stop the old server**: `ENDSBS JETTY *IMMED` (or `JETTY/ENDJTYSVR`). Check with `WRKACTJOB SBS(JETTY)` that the job has ended.
4. **Install 26.2** as described in [Pre-packaged JETTY IBM i installation](#pre-packaged-jetty-ibm-i-installation), with:
    - a **new installation directory**, for example `/home/jetty17`,
    - a **new installation library**, for example `JETTY17`,
    - the **same HTTP port** as before, so clients do not have to change their settings,
    - the same Jetty user as before (the existing profile is reused).
5. **Start and verify** the new server: `STRSBS JETTY17/JETTY17`, then the checks listed in the installation steps.
6. **Re-apply your settings**, only if you had changed them. Do not copy the old `.ini` files: Jetty 12 uses a different format. Restart the server after each change.
    - HTTPS: see [Enabling HTTPS](#enabling-https-26-2-and-later). Copy your keystore to a folder outside the installation directory first.
    - Logging: apply your changes to `/home/jetty17/base/resources/log4j2.xml`.
7. **Update your startup program** (`QSTRUP`): replace `STRSBS JETTY/JETTY` with `STRSBS JETTY17/JETTY17`.
8. **Remove the old server** once you no longer need to go back: `DLTLIB JETTY`, then `rm -rf /home/jetty` in QShell.

To go back to the Java 8 version before step 8: `ENDSBS JETTY17 *IMMED`, then `STRSBS JETTY/JETTY`.

### Pre-packaged JETTY Windows installation
Elias Webservices 26.2 and later runs on **Java 17** (bundled with the installer, nothing to install separately) and Jetty 12.
1. Run `Setup_Elias-Webservices-X.Y.Z_Win64.exe` as administrator.
2. The wizard asks for the **service account** and the **HTTP port** the server listens on. Use the same port your clients (VS Code, CI) connect to.
3. When the installer finishes, the `JettyService` Windows service starts automatically.
4. Verify the installation:
    - In `services.msc`, `JettyService` shows `Running`.
    - The last lines of `<install folder>\base\logs\jetty.log` contain `Started ... Server` and your port.
    - `http://localhost:<port>/elias/arcad/v1/ping` returns a short JSON answer (no login needed). Then connect from VS Code.

HTTPS is disabled after installation. See [Enabling HTTPS](#enabling-https-26-2-and-later).

### Upgrading a Windows server from a Java 8 version (2.6.x) to 26.2
An **in-place upgrade from a Java 8 version is not supported on Windows**: running the 26.2 installer over an existing Java 8 installation produces a mixed installation that cannot start. Uninstall the old server and install the new one instead.

Your data is not affected. The Elias REST API Server is a middleware between your clients and the IBM i: custom actions, projects and everything else it works with are stored on the IBM i, not on the Windows server. After the upgrade, users have to sign in again.

```
Back up -> Stop service -> Uninstall -> Delete leftover folder -> Install 26.2 -> Verify -> Re-apply settings
```

1. **Back up** to a folder outside the install folder (for example `C:\Backup\Elias-Java8`):
    - the whole `start.d` folder (HTTP port, HTTPS settings),
    - your HTTPS keystore file and its password, if you use HTTPS (it holds your certificate and private key and cannot be recreated),
    - `resources\log4j2.xml`, if you changed the logging settings,
    - anything you added to `webapps` other than `elias.war`,
    - optionally the `logs` folder.

    Also note the **HTTP port** and the **service account** (`services.msc` > `JettyService` > Properties > Log On). Keep the old installer, in case you need to go back.

    > Do not copy the backed-up files back into the new installation. Jetty 12 uses a different configuration format (for example the module line in `http.ini` is written differently, and the old `server.ini` refers to modules that no longer exist). Use the backup as a reference to re-enter your settings. Only the HTTPS keystore and `log4j2.xml` can be reused as they are.
2. **Stop** `JettyService` in `services.msc` and wait until it shows `Stopped` (this can take a few minutes).
3. **Uninstall** *ARCAD-Elias Webservices* from *Settings > Apps > Installed apps* (or run `unins000.exe` in the install folder). The uninstaller removes the Windows service. Do not just delete the folder: the service and the installer registration would stay behind, and the new installer would treat the machine as an upgrade.
4. **Delete the whole old install folder**, for example `C:\Program Files\ARCAD Solutions\Jetty` (save the logs you want to keep first). The installer never overwrites a file that already exists, so any leftover file would silently stay in the new installation. If Windows reports a file in use, check that the service is gone and that no `java.exe` or `prunsrv.exe` from that folder is running in Task Manager.
5. **Install 26.2** as described in [Pre-packaged JETTY Windows installation](#pre-packaged-jetty-windows-installation), with the **same service account and the same HTTP port** as before.
6. **Re-apply your settings**, only if you had changed them in the old server. Restart the service after each change.
    - Port: `<install folder>\base\start.d\http.ini`, line `jetty.http.port=`.
    - HTTPS: see [Enabling HTTPS](#enabling-https-26-2-and-later).
    - Logging: apply your changes to `<install folder>\base\resources\log4j2.xml`.

To go back to the Java 8 version, uninstall 26.2, delete the install folder, run the old installer and restore your backed-up files (they match the old server's format).

### Enabling HTTPS (26.2 and later)
1. Copy your keystore file to a folder **outside** the install folder (for example `C:\ARCAD\certs`), so it survives future reinstalls.
2. In `<jetty_base>/start.d` (`<install folder>\base\start.d` on Windows), rename `https.ini.off` to `https.ini` and set (remove the leading `#` where needed):
```
jetty.ssl.port=8443
jetty.httpConfig.securePort=8443
jetty.sslContext.keyStorePath=C:/ARCAD/certs/mykeystore.p12
jetty.sslContext.keyStoreType=PKCS12
jetty.sslContext.keyStorePassword=<your keystore password>
jetty.sslContext.keyManagerPassword=<your key password, if different>
```
Use `keyStoreType=JKS` for a JKS keystore, and forward slashes in the path.
3. Restart Jetty and check `logs/jetty.log`. A wrong path or password prevents the server from starting.

### Enabling log4j2 logging on Jetty
> Not needed with Elias Webservices 26.2 and later (Jetty 12): log4j2 is enabled by default. This section applies to older, Java 8 based installations only.

Elias uses log4j2 for logging, but support for l4j2 may not be enabled in Jetty, as it was recently added in Jetty pre-packaged setup. Here is how to enable it manually **if elias.log does not show up under Jetty's logs folder**:
1. Stop Jetty
2. Open a command terminal and go to Jetty's installation folder
3. Run the following command: `java -jar start.jar --add-to-start=logging-log4j2`
4. Once the above command is completed, open the `<jetty_base>/resources/log4j2.xml` file and replace its content with the one below.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" name="Jetty">
    <properties>
        <property name="logging.dir">${sys:jetty.logging.dir:-logs}</property>
    </properties>

    <Appenders>
        <Console name="console" target="SYSTEM_ERR">
            <PatternLayout>
                <Pattern>%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n</Pattern>
            </PatternLayout>
        </Console>

        <RollingRandomAccessFile name="file"
            fileName="${logging.dir}/jetty.log"
            filePattern="${logging.dir}/jetty-%d{MM-dd-yyyy}.log.gz"
            ignoreExceptions="false">
            <PatternLayout>
                <Pattern>%d [%t] %-5p %c %x - %m%n</Pattern>
            </PatternLayout>

            <Policies>
                <TimeBasedTriggeringPolicy />
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
        </RollingRandomAccessFile>

        <RollingRandomAccessFile name="elias"
            fileName="${logging.dir}/elias.log"
            filePattern="${logging.dir}/elias-%i.log.gz"
            ignoreExceptions="false">
            <PatternLayout>
                <Pattern>%d [%t] %-5p %c %x - %m%n</Pattern>
            </PatternLayout>

            <Policies>
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
        </RollingRandomAccessFile>
    </Appenders>

    <Loggers>
        <Root level="info">
            <AppenderRef ref="file"/>
        </Root>
        
        <Logger name="com.arcadsoftware.elias" level="info" additivity="false">
            <AppenderRef ref="elias" />
        </Logger>
    </Loggers>
</Configuration>
```

## Elias CLI
The Elias CLI comes packaged as an `.rpm` file. It required if you plan on developing your applications using Project mode. It must be installed on the development IBM i server. It can be download from [ARCAD's Customer Portal](https://portal.arcadsoftware.com/).

### Installation
1. Copy the `eliasCLI-X.Y.Z.ibmi7.3.ppc64.rpm` on the IFS, in the `/tmp` folder.
2. Open a QShell session or connect using SSH
3. Install or update Elias CLI using yum: `yum install /tmp/eliasCLI-X.Y.Z.ibmi7.3.ppc64.rpm`