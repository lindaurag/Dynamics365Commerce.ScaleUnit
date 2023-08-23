 [[_TOC_]]

This folder contains necessary instrumentation scripts and sample code to support smooth development experience creating Commerce Scale Unit (CSU) extensions.

# Introduction

CSU developer experience is supported by Visual Studio and Visual Studio Code IDEs. The Visual Studio Code is preferable since it supports automated deployment of Base Scale Unit and an Extension.
On-prem Scale Unit's Base Deployment Package is downloadable from LCS and is created by Microsoft. The Extension Package is created by third-parties while compiling the code in this repository. Development process involves installing the Base package first and then the Extension package is installed every time a new version of an extension should be debugged/tested. Both of the deployed packages, once installed, can be seen via the Windows feature *Add or remove programs*.

This process requires minimal amount of orchestrating [scripts](./Scripts) which mainly execute the above mentioned Deployment Packages with required parameters. The purpose of the scripts is only to provide fast and easy development experience and should not be used in any production deployment because they might be changed/removed at any time without a notice. 

This process doesn't require a specific pre-configured environment or virtual machine. Development and testing can be done on any machine with relatively modern version of Windows. If you don't require Modern POS development you can leverage Windows 10, Windows Server 2016/2019. If you require Sealed Modern POS development, install [these](https://docs.microsoft.com/en-us/dynamics365/commerce/dev-itpro/pos-extension/pos-extension-getting-started#set-up-the-pos-development-environment) prerequisites.

The process deploying CSU in your environment utilizes [Sealed Installer packages and frameworks](https://docs.microsoft.com/en-us/dynamics365/commerce/dev-itpro/enhanced-mass-deployment). Developers will experience a very similar process of installing the Scale Unit and its extensions in their local development environments, therefore reducing the potential for unexpected installation problems in production.

If you choose to develop by leveraging fully setup CSU locally with all actual components, the data synchronization between Channel and Headquarters (HQ) databases involves additional [set of steps](https://docs.microsoft.com/en-us/dynamics365/commerce/dev-itpro/retail-store-scale-unit-configuration-installation#configure-a-new-commerce-scale-unit) which is used in production systems for On-Prem Scale Units.


# Prerequisites

## Build-time

1. Clone https://github.com/microsoft/Dynamics365Commerce.ScaleUnit
1. Install 64 bit version of VS Code for Windows from https://code.visualstudio.com/download
1. Install *.Net Core SDK 3.1* for Windows x64 from https://dotnet.microsoft.com/download/dotnet/3.1.
1. Install *.NET Framework 4.7.2 Developer pack* from https://dotnet.microsoft.com/en-us/download/dotnet-framework/thank-you/net472-developer-pack-offline-installer.
1. Install the *Hosting Bundle* (click literally "Hosting Bundle" link, not "x64" nor "x86") for Windows from the same above referenced link.
1. Navigate to https://lcs.dynamics.com/V2/SharedAssetLibrary select the section *Retail Self-service package files* and then locate there the file ending with *Commerce Scale Unit (SEALED)*. Make sure to select there the version for the release you need, for instance 10.0.22, 10.0.23 and so on. Download the file and place it in the folder [Download](./Download)
1. Launch the VS Code as Administrator and open src\ScaleUnitSample by leveraging the menu File->Open **Folder**.
1. Once VS Code is asking to install recommended extensions - proceed installing them, the set includes *"C# for Visual Studio Code (powered by OmniSharp)"* and might include other extensions as well. If you see a message indicating that .NET Core SDK could not be located,  select the suggested option to get the SDK and follow the steps to install it.
1. In VS Code click Terminal->Run Task...->build-extension:
   - If there is an error indicating running scripts is disabled on this system, then open cmd.exe as Administrator and execute ```powershell Set-ExecutionPolicy RemoteSigned``` then restart VS Code. Do not change the ExecutionPolicy on production machines unless you understand [the implications of changing security policies.](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies).
   - If there is an error indicating the SDK 'Microsoft.NET.Sdk' specified could not be found, then navigate to https://visualstudio.microsoft.com/downloads/, scroll down to "Tools for Visual Studio", expand it and download/run "Build Tools for Visual Studio". In the installation wizard, go to "Individual components"->Type **.NET SDK**-> Check the CheckBox **.NET SDK** and hit "Install".
   - If there is an error "*The build task could not find node.exe which is required to run the TypeScript compiler. Please install Node and ensure that the system path contains its location.*" install Windows Installer 64 bit [Node.JS](https://nodejs.org/en/download/current/), when prompted set the checkbox "Automatically install the necessary tools".

## Run-time
### Shared across Self-Hosted and IIS-Hosted modes
The below prerequisites are required at runtime for both modes: Self-Hosted and IIS-Hosted.

1. Channel DB is deployed in both modes therefore an instance of [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) is required. The easiest way is to have [default](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/database-engine-instances-sql-server?view=sql-server-ver15#instances), rather than Named, instance of [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads). You can use any edition, even Express, minimal supported version is 13.0.5026.0 [SqlServer 2016 SP2](https://docs.microsoft.com/en-US/troubleshoot/sql/general/determine-version-edition-update-level). If you don't have Default instance of SQL Server installed then the deployment will fail with error indicating an instance could not be located, if you want to use Named instance instead then modify the file [tasks.json](./.vscode/tasks.json) by inserting this line after \#16:
 ```"baseProduct_SqlServerName" : "PutYourSqlServerSeenInSSMSHere",```. Substitute **PutYourSqlServerSeenInSSMSHere** with your SQL Server Name.

   Number of features must be enabled in the SQL Server instance:
   - Mixed (SQL + Windows/Integrated) authentication must be configured.
   - The feature *Full Text Search* must be enabled by following \#8 and associated steps described [here](https://docs.microsoft.com/en-us/sql/database-engine/install-windows/add-features-to-an-instance-of-sql-server-setup?view=sql-server-ver15#to-add-features-to-an-instance-of-), it is available in all editions starting SQL Server Express.

   If you don't have SQL Server installed yet, follow these steps:
   - Navigate [here](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) and click Download against Express version.
   - Select "Custom" for the Installation Type
   - Hit "Install"
   - Once the installation completes click "Installation" link on the left hand side and then click "New SQL Server stand-alone installation or add features to an existing installation"
   - Follow the wizard until you reach the screen "Feature Selection", you don't have to install "Machine Learning Services" and all its subcomponents but "Full-Text and Semantic Extractions for Search" must be selected.
   - Hit "Next" and at the screen "Instance Configuration" select "Default Instance". Hit Next.
   - At the screen "Database Engine Configuration" select "Mixed Mode (SQL Server authentication and Windows authentication)" and create a password for the Server Administration account. Continue the wizard to the completion.

### IIS-Hosted mode
1. Internet Information Services (IIS) with a number of features turned on, the installer will report any required but not yet enabled features. To proactively install IIS prerequisites navigate to 'Turn Windows features on or off' and then:
   - Under *.NET Framework 3.5.X*, ensure that the *Windows Communication Foundation HTTP Activation* option is selected.
   - Under *.NET Framework 4.X* navigate to *WCF Services* and ensure *HTTP Activation* is selected.
   - Under *Internet Information Services* enable these:
     -  Web Management Tools -> IIS 6 Management Compatibility (IIS Metabase and IIS 6 configuration compatibility)
     -  Web Management Tools -> IIS Management Console
     -  World Wide Web Services -> Application Development Features (.NET Extensibility, ASP.NET 3.5/4.8, ISAPI Extensions, ISAPI Filters)
     -  World Wide Web Services -> Common HTTP Features (Static Content, Default Document, Directory Browsing, HTTP Errors)
     -  World Wide Web Services -> Health and Diagnostics (HTTP Logging, Request Monitor)
     -  World Wide Web Services -> Performance Features (Static Content Compression)
     -  World Wide Web Services -> Security (Windows Authentication, Request Filtering)
1. Only TLS 1.2 (not TLS 1.0/1.1) should be enabled in the system, you can leave this prereq check to the installer which will print out actionable instructions in case your machine needs changes in that area. Or you can proactively configure strong cryptography as described [here](https://docs.microsoft.com/en-us/mem/configmgr/core/plan-design/security/enable-tls-1-2-server#configure-for-strong-cryptography) and make sure TLS 1.0/1.1 is disabled by following [this](https://docs.microsoft.com/en-us/windows-server/security/tls/tls-registry-settings#tls-dtls-and-ssl-protocol-version-settings) section. In case your registry will not have subkeys TLS 1.0 (and 1.1) ->Client/Server - you would need to manually create them.


#Folder structure
- ChannelDatabase - contains the set of SQL files which will be executed against Channel DB during the Extension deployment. The scripts are run in alphabetical order, see details [here](https://docs.microsoft.com/en-us/dynamics365/commerce/dev-itpro/channel-db-extensions)
- CommerceRuntime - the set of CommerceRuntime (CRT) and CSU files composing CRT/CSU Extension.
- Download -  the container for the Base Sealed Scale Unit installer. Its subfolder ChannelData will contain an automatically downloaded package with Demo Data which will be applied against the Channel DB during deployment.
- E-CommerceProxyGenerator - contains the set of files required to generate a proxy capable of making CSU calls.
- Installer - the project responsible for the Extension Installer generation used for on-prem Scale Unit Deployment.
- POS - contains samples demonstrating how to create POS extensions.
- ScaleUnit - the project responsible for the generation of the Extension package used for cloud Scale Unit Deployment.
- Scripts - the set of internal scripts which help facilitating development experience.

#Components' outputs
Once the build is successfully completed, you can leverage the outputs to deploy your extensions:
- Cloud Scale Unit extension package is located in ScaleUnit\bin\Debug\netstandard2.0 folder - used for Cloud deployments
- Scale Unit extension installer is located in Installer\bin\Debug\net472  - used for on-prem (in-store) installations 

#Test and Debug

## Self-hosted mode
This is a very light-weight version of the scale unit providing a quick way to develop wide spectrum of CRT/RTS extensions requiring minimal to no dependencies on other systems/services.

### Pros
- Doesn't require HQ.
- Doesn't require IIS. Retail Server is hosted in a Console Application
- TLS configuration is not required
- RTS communication is mocked via a set of Demo Mode CRT Handlers
- Async Client is not needed, the Channel DB is automatically populated with a Demo Data.
- Requires very few parameters to initiate the Scale Unit deployment so the required prerequisites are minimal. For instance - no certificates have to be explicitly created and provided to the installer.
- Provides very fast and easy introduction into CRT/Retail Server development and runtime capabilities.

###Cons
- Doesn't match topology used in real production environments (no CPOS, no Async Client, no RTS calls, no HTTPS).
- Real Time operations cannot be fully tested (they must be mocked).
- Device Activation is not supported, therefore this mode cannot be used for end to end usage with MPOS/CPOS/StoreCommerce.
- Only those APIs which work in Anonymous authentication context can be used.

### Debugging
Self-hosted mode is enabled by default, to run it hit *F5 (Start Debugging)* (supported in Visual Studio Code only), then the following will be done automatically (supported in Visual Studio Code only):
- The Scale Unit Sample code is compiled. This essentially means an extension compilation which results in a package containing Sealed CSU Extension installer.
- The Base Sealed Scale Unit installer is deployed if it was not deployed yet.
- The package containing Demo Data is downloaded from the SDK's feed and then being applied to the Channel DB
- Sealed Scale Unit Extension installer is deployed
- Web browser page with a result of a call to CSU's health-check endpoint is rendered
- The debugger is attached to the process hosting the CSU and the CSU is ready to serve incoming requests at the Url http://localhost:12345.

While the local scale unit is serving the requests, you can observe "DEBUG CONSOLE" to watch its internal diagnostics logging in real-time which can be helpful while debugging.

#### Switching from IIS mode to Self-Hosted mode
If you will switch between IIS and Self-Hosted modes make sure to do the below steps (if this is your first run - you can skip them because those settings are default ones):
- In VS Code hit the button *Run and Debug (Ctrl+Shift+D)*, a Drop Down Menu with a green arrow will be displayed under the top navigation bar, select there *Debug with Self-Host* 
- Open the file [.vscode/tasks.json](./.vscode/tasks.json) and make sure *baseProduct_UseSelfHost* is set to *true*.

## IIS mode
This is a complete on-prem Scale Unit with all the components matching real production deployments where nothing is mocked. 

### Pros
- Represents fully capable on-prem Scale Unit hosting Retail Server in IIS, employs real (not mocked) RTS interaction with HQ as well as involves Async Client populating Channel DB based on HQ DB data as regular Scale Unit does so no Demo Data is involved.

### Cons
- Requires additional steps for creating certificates, AAD registrations and downloading the configuration file from HQ.
- Might be considered as a "heavy" approach if a simple part of the development needs to be done.

### Debugging
1. In VS Code hit the button *Run and Debug (Ctrl+Shift+D)*, a Drop Down Menu  with a green arrow will be displayed under the top navigation bar, select there *Debug with IIS*
1. Make the following changes in the file [.vscode/tasks.json](./.vscode/tasks.json) 
   - *baseProduct_Port* - you can keep the default port 446 or change it to any value you want. If you specify the port used by another application, the Base Installer deployment will detect that and ask you to provide another port.
   - *baseProduct_SslCertFullPath* should point to the SSL certificate used by CSU Web Site to be capable of HTTPS communication. For this non-Production setup the certificate can be a Self-Signed one and it  must be stored in LocalMachine/Personal certificate store. To generate the certificate follow the steps:
      1. Open IIS and double-click the icon representing your machine name
      1. Double-click *Server Certificates* icon
      1. On right hand side click the link *Create Self-Signed Certificate...*
      1. A popup will be opened and will ask you to provide a friendly name of the certificate. Specify there Fully Qualified Domain Name (FQDN) of your machine which includes the machine name itself as well as a domain if it is joined into one. You can find out FQDN by executing the Powershell ```[System.Net.Dns]::GetHostEntry("")```. FQDN can also be retrieved from the System widget of the Windows Control Panel, look there into the field *Full computer name*.
      1. Keep the default value for the certificate store - *Personal*
      1. Once you click *OK* button, find just created certificate in the list and double click it
      1. Select the tab *Details* and locate a Thumbprint there
      1. Copy/paste the thumbprint into a text editor, make it uppercased and specify it at the end of the predefined template for the option *baseProduct_SslCertFullPath* so it should look similar to: store:///My/LocalMachine?FindByThumbprint=**YourThumbprintGoesHere**

      Note: the Base Installer requires 2 more certificates, so there are 3 certificates in total. For all production deployments 3 different certificates should be created for security reasons but for this dev setup you may decide to save time and use the same certificate for all 3 configuration options if this is not against your policies. In this case you can provide the same thumbprint in the 2 options described below.
   - *baseProduct_RetailServerCertFullPath* In the way similar to the above, you need to update this parameter with the thumbprint of the certificate you will create. This certificate will be used to represent Retail Server's Identity, which is used, for instance, when CSU issues security tokens used in POS scenarios. To create this certificate follow the same process as for the above SSL certificate but specify any FriendlyName you want, for instance, RsTestIdentity. You will need this certificate later while setting up AAD application.
   - *baseProduct_AsyncClientCertFullPath* This certificate is used by Async Client when it acquires a security token from AAD for a sake of communication with HQ. Follow the same approach as above to create the certificate and similarly name it as you want, for instance, *AsyncClientTestIdentity*.
   - *baseProduct_RetailServerAadClientId* AAD Application Client ID representing Retail Server's identity which is used, for instance, when CSU issues security tokens used in POS scenarios. Follow the steps under \#3 described [here](https://community.dynamics.com/ax/b/axforretail/posts/how-to-point-cpos-to-use-your-own-azure-ad-application) to create the application and retrieve its *Application (client) ID*.
   - *baseProduct_RetailServerAadResourceId* This is the resource ID of the above registered AAD application. Supply here the value described as *Application ID URI* in 3c [here](https://community.dynamics.com/ax/b/axforretail/posts/how-to-point-cpos-to-use-your-own-azure-ad-application).
   - *baseProduct_CposAadClientId* This is AAD Application Client ID representing your Cloud POS. Follow the step \#4 [here](https://community.dynamics.com/ax/b/axforretail/posts/how-to-point-cpos-to-use-your-own-azure-ad-application) to create the application and retrieve its *Application (client) ID*. This completes CSU/CPOS setup in AAD, make sure to follow the step \#6 from [here](https://community.dynamics.com/ax/b/axforretail/posts/how-to-point-cpos-to-use-your-own-azure-ad-application) to complete required setup in HQ.
   - *baseProduct_AsyncClientAadClientId* This is the AAD Application Client ID used by Async Client when it needs to authenticate with HQ. To create this application register one more AAD application by following 3a-3b from [here](https://community.dynamics.com/ax/b/axforretail/posts/how-to-point-cpos-to-use-your-own-azure-ad-application). To register just created application with HQ follow \#2 from [here](https://community.dynamics.com/ax/b/axforretail/posts/service-to-service-authentication-in-ax7)
   - *baseProduct_Config* Specify the file name, without a full path, corresponding to the Channel Database Configuration downloadable from HQ as described in the bullet 4 of the section [Download the CSU installer](https://docs.microsoft.com/en-us/dynamics365/commerce/dev-itpro/retail-store-scale-unit-configuration-installation#download-the-commerce-scale-unit-installer). Once you download the file from HQ place it in the [Download](./Download) folder.
   - *baseProduct_UseSelfHost* Set to *false*.

1. Hit F5. As in case of the Self-Hosted mode the Base Installer will be deployed first and that will be followed by a deployment of the Extension Installer.
1. While Base Installer will execute it will perform number of prerequisite checks and report any actions required, the prerequisites verify: SQL Server, IIS, TLS, .Net Core Hosting Bundle, and so on.
1. Once the Base Scale Unit and the Extension packages are deployed, VS Code will open a browser window with the results of a call to health-check and will also display a dialog asking to attach a debugger. Attaching is optional, if you want to attach type *w3wp* in the list of processes and select the row which will contain *RssuCore* in it, that is the name of the IIS Application Pool used to run CSU Web Site.
1. To experience the debugging process in action, put a breakpoint inside the method [SimplePingGet](./CommerceRuntime/Controllers/UnboundController.cs) and, assuming you are debugging with in a Self-Hosted mode, navigate in the browser to http://localhost:12345/Commerce/SimplePingGet, you will see the breakpoint is hit.

At this time you have fully functional On-Prem deployed Scale Unit which includes:
- Channel DB
- Async Client
- ASP.NET Core 6 based Retail Server capable of interacting with HQ via RTS
- Cloud POS

You can find URLs corresponding to just deployed CPOS and CSU if you review the Base Installer's log towards the end when CSU and CPOS are health-checked. In order to populate the Channel DB with data from HQ you will need to execute the step \#28 from [here](https://docs.microsoft.com/en-us/dynamics365/commerce/dev-itpro/retail-store-scale-unit-configuration-installation) assuming you have already performed previous steps described in that document.

# Switching between Self-Hosted and IIS modes
## Switching to Self-Hosted mode
- Hit the button *Run and Debug (Ctrl+Shift+D)*, a Drop Down Menu with a green arrow will be displayed under the top navigation bar, select there *Debug with Self-Host* 
- Open the file [.vscode/tasks.json](./.vscode/tasks.json) and make sure *baseProduct_UseSelfHost* is set to *true*.

## Switching to IIS mode
- Hit the button *Run and Debug (Ctrl+Shift+D)*, a Drop Down Menu with a green arrow will be displayed under the top navigation bar, select there *Debug with IIS* 
- Open the file [.vscode/tasks.json](./.vscode/tasks.json) and make sure *baseProduct_UseSelfHost* is set to *false*.

# Miscellaneous
While using this Sample to compile your final extension package, don't forget to exclude all projects/files you don't really need in your Extension package. Otherwise you would end up shipping the samples as part of your extension package.

# Troubleshooting
If anything doesn't go as expected while the packages were deployed, review the verbose set of logs and associated messages in the *TERMINAL* tab. If you cannot figure out what is wrong on your own, then contact Microsoft to get help and provide:
- Verbose description of the actions performed
- The log file referenced at the very beginning and the very end of the Base Product's deployment process output.
- VS Code Terminal's output including warnings/errors.

Runtime logs corresponding to  Retail Server and Async Client (in case of IIS host) can be found in Event Viewer under *Windows Logs*->*Application*. Filter the log by the Sources:
- Microsoft Dynamics - Async Client Service
- Microsoft Dynamics - Retail Server

Self-Hosted CSU will also print runtime logs directly to the VS Code Terminal.

# Out Of the Box Tasks
A set of tasks is available via VS Code's *Terminal->Run Task...* to achieve various goals:
- *build-extension* - builds an extension.
- *check-dotnet-sdk* - reveals .NET Core SDK version available for Visual Studio Code.
- *check-ps-bitness* - checks if PowerShell available for Visual Studio Code is of required 64 bit version.
- *clean-extension* - cleans build output generated while the extension was built.
- *install* - verifies all Dev Time prerequisites and then performs all required actions for the selected mode (Self-Hosted vs IIS) to deploy the Scale Unit ready for a debug session. This includes installation of the Base Scale Unit as well as Extension Installer.
- *uninstall* - uninstall an Extension and Base Scale Unit.
- *uninstall-base-product* - uninstalls Base Scale Unit. Will fail if it detects an installed Extension.
- *uninstall-extension* - uninstalls an Extension.

# Feedback
If anything, in regards to the Scale Unit Dev Experience, is not working as expected or you need help, reach out via:
- https://github.com/microsoft/Dynamics365Commerce.ScaleUnit/issues
- https://community.dynamics.com/365/commerce/f/dynamics-365-commerce-forum
- https://www.yammer.com/dynamicsaxfeedbackprograms/#/threads/inGroup?type=in_group&feedId=1585934