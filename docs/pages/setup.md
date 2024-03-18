# Setup
## Get the files
You can get all the files mentionned below from:
- [ARCAD-Elias release on GitHub](https://github.com/ARCAD-Software/elias-vscode/releases/latest)
- [ARCAD's Customer Portal](https://portal.arcadsoftware.com/) (Go to the `Products` section and open the `ARCAD Skipper` section.

## Elias REST API Server
The Elias REST API Server comes packaged as a `.war` file. It can run on any operating system that supports Java. It must be deployed on an application server (e.g. IBM i integrated Web Application Server, Jetty, Tomcat...whatever floats your boat!), under the `/elias` context.
A pre-packaged, ready to use Jetty installation is available too. It contains a Jetty runtime packaged with `elias.war`.

### Pre-packaged JETTY IBM i installation
1. Copy the `Setup_Elias-Webservices-X.Y.Z_IBMi.jar` on the IFS, in the `/tmp` folder.
2. Open a QShell session or connect using SSH
3. Go to the `/tmp` folder and start the installation process
```bash
cd /tmp
java -jar Setup_Elias-Webservices-X.Y.Z-SNAPSHOT_IBMi.jar
```
4. The installation process will ask for several installation values, with default values between brackets. Press ENTER to accept the default values or enter the desired value then press ENTER.
5. After the installation, start the JETTY subsystem to start JETTY web application server. For example, if you installed JETTY in the default `JETTY` library: 
    - `STRSBS JETTY/JETTY`
6. Alternatively, JETTY can started and stopped using the following commands:
    - `JETTY/STRJTYSVR`
    - `JETTY/ENDJTYSVR`

### Enabling log4j2 logging on Jetty
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