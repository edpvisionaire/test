# Morph Web Deployment (Wild Fly Server) for Windows

### Required Files / Installers

                
1. Java Development kit – **jdk-8u202-windows-x64**
2. Wildfly version: **26.1.1**
3. **laragon** or **xampp**
4. JDBC connector for postgres database – **postgresql-42.5.4.jar**
5. Copy of **ProdigenesisSecurity_2.war** and **ProdigenesisMain.war**
6. A copy of Morph front end - [Morph Front End Repository](https://vault.prodigenesis.com/visionaireweb/morphweb-frontend.git)
7. A copy of Morph back end - [Morph Back End Repository](https://vault.prodigenesis.com/visionaireweb/morphwebbackend.git)
                

### Configure Java Runtime Environment
                
- Install Java Development kit to default location:  **C:\Program Files\Java\jdk1.8.0_202**
- Add a new System Variable called **JAVA_HOME** with value pointing to JDK installation directory: **C:\Program Files\Java\jdk1.8.0_202**

![SCREENSHOT 1](instructions/screenshot1.png?raw=true "Java home system variable")

- Add the newly added environment variable **(JAVA_HOME)** to system path.

![SCREENSHOT 2](instructions/screenshot2.png?raw=true "Java home system path")
![SCREENSHOT 3](instructions/screenshot3.png?raw=true "Java home system path")

- Verify that Java Environment is working by going to command prompt, type in **java** then press enter.

![SCREENSHOT 4](instructions/screenshot4.png?raw=true "Confirm java runtime")

                 

### Configure Java Runtime Environment

- Extract **wildfly zip** file to root directory of local **drive C:**
**C:\wildfly-26.1.1.Final **
- Navigate to **C:\wildfly-26.1.1.Final\bin** folder and open **standalone.conf.bat** to any text editor of your choice.

![SCREENSHOT 5](instructions/screenshot5.png?raw=true "Standalone file configuration")

- find the line of code **"x%JBOSS_JAVA_SIZING%"**. The set of codes mention here controls the heap size memory limit that wildfly use. Make sure to check the lines of code to be the same. If not change it accordingly.

Here is the code for reference:
```
if "x%JBOSS_JAVA_SIZING%" == "x" (
    rem # JVM memory allocation pool parameters - modify as appropriate.
    set "JBOSS_JAVA_SIZING=-Xms64M -Xmx6G -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Dresteasy.preferJacksonOverJsonB=true -XX:+UseConcMarkSweepGC"
)
```
- Navigate to **C:\wildfly-26.1.1.Final\standalone\configuration** and open **standalone.xml** to any text editor of your choice.

![SCREENSHOT 6](instructions/screenshot6.png?raw=true "Standalone XML configuration")

- find the line of code **<interfaces>**. Change the **inet-address** ip to **0.0.0.0** This will allow access to wildfly server via the server ip address using port **8080**.

![SCREENSHOT 7](instructions/screenshot7.png?raw=true "Interface")

- navigate to **C:\wildfly-26.1.1.Final\bin** and start wildfly server by running **standalone.bat**. This will popup a command window listing out the server status and the admin url for wildfly dashboard. You can access wildfly dashboard by going to **http://127.0.0.1:9990** or by **http://{ip_address_of_server}:9990**

![SCREENSHOT 8](instructions/screenshot8.png?raw=true "Standalone bat file location")

![SCREENSHOT 9](instructions/screenshot9.png?raw=true "Wildfly console")

- On Wildfly dashboard, go to **Deployment** menu and select upload deployment

![SCREENSHOT 10](instructions/screenshot10.png?raw=true "Wildfly dashboard")

- Upload **postgresql-42.5.4.jar** JDBC connector. Upon successful deployment, **postgresql-42.5.4.jar** should be in the list of Deployment.

![SCREENSHOT 11](instructions/screenshot11.png?raw=true "JDBC Connector")

- In Wildfly dashboard go to **Configuration → subsystems → datasources & drivers** and add a new data source

![SCREENSHOT 12](instructions/screenshot12.png?raw=true "Add data source")

- On data source window,
step 1: Select template → PostgreSQL
step 2: attributes → name: ProdigenesisSecurityDS 
            attributes → JNDI Name: java:/ProdigenesisSecurityDS
step 3: JDBC driver → name: postgresql-42.3.5.jar
step 4: Connection → connection url: {provide connection url to security database}
			Connection → username: {provide username for security database}
			Connection → password: {provide password for security database}
Step 5: click on **test connection** button and click **next** to finish adding data source.

Note: in the case of test connection failure, double check the connection tab for any invalid configuration

- Add another data source 
tep 1: Select template → PostgreSQL
step 2: attributes → name: ProdigenesisMain2DS 
            attributes → JNDI Name: java:/ProdigenesisMain2DS
step 3: JDBC driver → name: postgresql-42.3.5.jar
step 4: Connection → connection url: {provide connection url to security database}
			Connection → username: {provide username for security database}
			Connection → password: {provide password for security database}
Step 5: click on **test connection** button and click **next** to finish adding data source.

Note: in the case of test connection failure, double check the connection tab for any invalid configuration

-  On Wildfly dashboard, go back to **Deployment** menu and **select upload** deployment

![SCREENSHOT 13](instructions/screenshot13.png?raw=true "Upload deployment")

- Upload the war files **ProdigenesisSecurity_2.war** and **ProdigenesisMain.war**

### Setting up Front End
#### Using xampp
1. install **xampp** to default directory **C:\xampp**
2. navigate to **C:\xampp\htdocs** and put morph folder (coming from the front end repository) inside
3. Run **xampp control panel** and start **apache server**

#### Using Laragon
1. install **laragon** to default directory **C:\laragon**
2. navigate to **C:\laragon\www** and put the morph folder (coming from the front end repository) inside
3. Run **Laragon**

### Configure file Attachment 
#### Option 1:
1. create a hidden shared distributed file system folder named **morphattachment$**
2. navigate to **C:\wildfly-26.1.1.Final\bin**, locate **jboss-cli.bat** and run it as administrator
3. type **connect** to login to wildfly cli
4. replace the **ip address** before running the commands

```
subsystem=undertow/configuration=handler/file=morphAttachements:add(directory-listing="false",path="//172.20.200.51/morphattachment$")

/subsystem=undertow/server=default-server/host=default-host/location=\/static:add(handler=morphAttachements)
```

#### Option 2:
1. make sure to wildfly server is running
2. locate **fileAttachmentInstaller.bat** file from inside and run it as an Administrator
3. the **fileAttachmentInstaller.bat** should prompt to enter **wildfly root directory** location and the **unc path** to use for file attachment storage

![SCREENSHOT 14](instructions/screenshot14.png?raw=true "fileAttachmentInstaller bat file")

the .bat file should automatically configure the settings to wildfly server.

Note: Make sure to restart wildfly server to take effect.

### Configure front end IP
- Navigate to **morph\resources\config.json** and edit the ip, filestorage and port configuration according to server ip address.

![SCREENSHOT 15](instructions/screenshot15.png?raw=true "Front end config json file")

Note: You can access morph web by visiting **{ip of sever}/morph/login/**


### Configure wildfly to run as window service
- Navigate to **C:\wildfly-26.1.1.Final\docs\contrib\scripts** and copy the **service** folder to **C:\wildfly-26.1.1.Final\bin**

![SCREENSHOT 16](instructions/screenshot15.png?raw=true "Service folder")

- Open the windows command prompt as an administrator
- Enter the following command to change your directory to the bin directory in your **WILDFLY** directory.
**CD YOUR_WILDFLY_HOME_DIR\bin\service**
- Enter the following command to install the service.
```
service install 
```
- Open **window services** and find **wildfly** from the list of service, change it property to automatically start.

Note: you may find a pdf format of this readme file in instructions folder of this repository
### 