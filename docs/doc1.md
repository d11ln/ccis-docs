---
id: doc1
title: Start here
sidebar_label: start
---



## Welcome!

Welcome to the National Climate Change Information System (NCCIS). <br />

The NCCIS is a first step towards the development of an open source, standards-based, and integrated portfolio of systems that aims to eliminate duplication of effort, limits multiplication of data sources, and is reusable on many levels of government in South Africa. 

This document aims to provide a cohesive guide to creating your own instance of the NCCIS with its coupled systems (NCCRD, NDAO, CSA - more on that later) and dependencies. For reference, the national, maintained system can be found [here](https://ccis.environment.gov.za). 

The tech stack used for this system: [C#](https://docs.microsoft.com/en-us/dotnet/csharp/) for the API, [ReactJS](https://reactjs.org/) for the client and [MS SQL](https://www.microsoft.com/en-us/sql-server/sql-server-2019) for the database.



## Environment

Please make sure you have the following installations on your machine: 

[Node.js](https://nodejs.org/en/download/) <br />
[npm](https://www.npmjs.com/) <br />
[ReactJS](https://reactjs.org/) <br />
[SQL Express 2017](https://www.microsoft.com/en-us/download/details.aspx?id=55994) <br />
[IIS](https://www.microsoft.com/en-za/download/details.aspx?id=2299) <br />
[Visual Studio](https://visualstudio.microsoft.com/) or [CMake](https://cmake.org/) (Optional)

## Server Requirements

Database: 
[SQL Server 2008 R2](https://www.microsoft.com/en-za/download/details.aspx?id=22985) (or later) <br />

Size: up to 10GB per database, times 10 contributing systems - allow for 100GB relational data <br />

[IIS v7](https://www.microsoft.com/en-za/download/details.aspx?id=2299) (or higher) <br />

Storage size: 10GB - 30GB <br /> 

[.Net Core Hosting Bundle v2.2.x](https://dotnet.microsoft.com/download/thank-you/dotnet-runtime-2.2.2-windows-hosting-bundle-installer)


## Server 

### Server Specs

Allocate a Windows server with the following specifications: <br />

```text
Windows Server 2016 
Processor: Intel(R) Xeon(R) CPU E5-2630 v2 @ 2.60GHz (4 processors) 
Installed memory (RAM): 16.00GB minimum, more preferred 
System type: 64-bit Operating System, x64-based processor 
Disk Storage: 500GB will be adequate, 1TB preferred 
```

### Setting up the server

Download and install [SQL Express 2017](https://www.microsoft.com/en-us/download/details.aspx?id=55994) <br />
Download and install [SSMS 18.4](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15#download-ssms)

#### Enable Internet Information Services (IIS)

In the `Server Manager`, navigate to:

  - `Add Roles and Features` (this will launch the Add roles and Features wizard),
  - in the menu pane, select `Server Roles`
  - ensure that `Web Server (IIS)` and all its dropdowns are selected
  - in the menu pane, select `Features`
  - ensure that the following is selected: `ISS Hostable Web Core`, `.Net Framework (and all its dropdowns)`
  - after that you will land on `Security` settings, select all and do the same for `IIS Management Tools` 
  - click `Install` to proceed. 

When prompted to do so, restart the server. 

#### Create your API folder

Navigate to `C:\inetpub\wwwroot\` and make a folder to house your API (we'll call it NCCRD) so that you have the path: `C:\inetpub\wwwroot\NCCRD\` <br />

In `C:\inetpub\wwwroot\NCCRD\` make two folders, `Client` and `API`

### Build your API

For the purpose of this walkthrough we'll be using Visual Studio, you're welcome to use other tools. 

This walkthrough assumes that you are building from the latest release (in this case the [NCCRD repo](https://github.com/SAEONData/NCCRD/releases)) <br />

  - Download and extract the [.zip](https://github.com/SAEONData/NCCRD/archive/v1.0.0.zip) file under the `Releases` section in github and open the project folder in Visual Studio
  - In the API root folder, create a `secrets.json` file that contains your connection string for SQL, it will look something like this: 

```
{
  "ConnectionStrings": {
    "NCCRD": "Data source=.; initial catalog=NCCRD; user id=YOUR_USER_ID; password=YOUR_USER_PASSWORD;"
  },
  "VmsApiBaseUrl": "path/to/VMS/API"
}
```
  - Build the solution (rightclick on the `.sln` file and run `Build`) 
  - In the solution explorer, rightclick on the project folder (in this case it's `NCCRD.Services.DataV2`) and click `Publish` <br />

You will now be presented with the Publish wizard where you should set the following configs: <br />

  - Set the deployment profile to `FolderProfile` and the `target location` to a folder you can easily locate  
  - Under `Summary`, 
    - set the `Configuration` to `Release` and 
    - set `Deployment Mode` to  `Self-contained`

Save your settings and publish! <br />

You will now have the API files populated inside the folder you published to, copy all its contents and paste it into the folder on your server that you had previously allocated for your API (in our case it's `C:\inetpub\wwwroot\NCCRD\API`) <br />



#### Configure IIS

Open IIS and in the `Connections` pane:
  - expand your server instance 
  - expand `Sites` 
  - if there is a default site, rightclick on it and delete it 
  - then rightclick on `Sites` and select `Add Website` 

This will open the `Add Website` wizard:

  - Under `Site name` enter the name of your web app (for the purpose of this walkthrough we'll call it `NCCIS`, this will be the main app from which the other apps branch off)
  - Under `Application pool` make sure `DefaultAppPool` is selected
  - In the `Content Directory` section set your `Physical path` to where your compiled files are stored, in this case it is the folder you created in the previous step: `C:\inetpub\wwwroot\NCCIS\Client`
  - Click on `Connect as...` and select `Application user`
  - Make sure that `start website immediately` is checked then click `OK` to complete the process

#### Connect your instance

Open SSMS and in the object explorer pane, rightclick on your server and select properties <br />

Allow SQL and Windows Auth <br />

Ensure that `Allow remote connections` is checked <br />

Continue to restarting your server.

## Database

### Restore backup

In SQL Server Manager, in the object explorer pane:
  - rightclick on Databases and select `Restore Database...`
  - under `Source`, select `Device` 
  - name your DB (in this case we'll call it `NCCRD`) 
  - under `Destination`, enter the path to your .bak file and click `OK`

To make sure the DB creation was a success, check for data in the DB folder (ie by running a query) <br />

#### Create a new login for your DB

If, for some reason, you need to create a new user for your database, do the following:

  - select username, password and set SQL Auth
  - under the option `Map to credentials`, select `all server roles` 
  - under `User mapping`, select `<your instance name> (in this case NCCRD)` 
  - select all database permissions 
  - under `Status`, ensure that `grant and enable login` is selected
  - click run 

## Client

### Preflight configuration

Get the source by forking or cloning this [repo](https://github.com/SAEONData/NCCRD.git)

Before you run the install and fire up your app you'll need to do some minor configuration to ensure everything is wired up for your instance, please follow the steps below to configure your instance.

In the App/JS directory, create a file called secrets.js and populate it with the following: 
```
import { ssoBaseURL } from './config/serviceURLs.js'
import { siteBaseURL } from './config/serviceURLs.js'

let config = {
client_id: 'NCCRD_React_Client',
  redirect_uri: `${siteBaseURL}#/callback#`,
  post_logout_redirect_uri: `${siteBaseURL}#/logout`,
  silent_redirect_uri: `${siteBaseURL}/silent_renew/silent_renew.html`,
  response_type: 'id_token token',
  scope: 'openid profile email SAEON_NCCRD_Web_API',
  authority: ssoBaseURL,
  automaticSilentRenew: true,
  filterProtocolClaims: false,
  loadUserInfo: true,
  silentRequestTimeout: 30000
}

export const userManagerConfig = config
```


### First run

cd into the client folder and run: 

```
npm i
```
This will take care of installing your dependencies. 

You'll want to watch for any warnings in the cli, any references to missing dependencies means that you'll have to install those yourself, for example: 

```
npm WARN @ant-design/icons-react@1.1.2 requires a peer dependency of @ant-design/icons but none is installed. You must install peer dependencies yourself. 
```
In the event that you come across any of these that hinder the installation or rendering of the app, simply run: `npm i` followed by the relevant dependencies as pointed out by the warnings.

If all went well and npm installed successfully, you can point your browser to the relevant port where you'll see an instance of the app. 

### Compile the client

Before you build your client, you'll want to configure the URLs and secrets file to match your requirements:

  - Setup your secrets.js file as per previous step, if you have not done so already
  - Set your URLs in app\js\config\serviceURLs.js so that the following reflects your configuration:

  ```
  if (CONSTANTS.DEV) {
   _apiBaseURL = 'your/dns/nccrd/api/odata/'
   _siteBaseURL = 'your/dns/nccrd/'
   _ndaoBaseURL = 'your/dns/ndao/api/odata/'
   _ndaoSiteBaseURL = 'your/dns/ndao/'
   _vmsBaseURL = 'https://ccis.environment.gov.za/vms/api/'
   _ssoBaseURL = 'https://identity.saeon.ac.za/'
   _mapServerBaseURL = 'https://ccis.environment.gov.za/map'
}

else if (CONSTANTS.TEST) {
   _apiBaseURL = 'your/dns/nccrd/api/odata/'
   _siteBaseURL = 'your/dns/nccrd/'
   _ndaoBaseURL = 'your/dns/ndao/api/odata/'
   _ndaoSiteBaseURL = 'your/dns/ndao/'
   _vmsBaseURL = 'https://ccis.environment.gov.za/vms/api/'
   _ssoBaseURL = 'https://identity.saeon.ac.za/'
   _mapServerBaseURL = 'https://ccis.environment.gov.za/map'
}

else if (CONSTANTS.PROD) {
   _apiBaseURL = 'your/dns/nccrd/api/odata/'
   _siteBaseURL = 'your/dns/nccrd/'
   _ndaoBaseURL = 'your/dns/ndao/api/odata/'
   _ndaoSiteBaseURL = 'your/dns/ndao/'
   _vmsBaseURL = 'https://ccis.environment.gov.za/vms/api/'
   _ssoBaseURL = 'https://identity.saeon.ac.za/'
   _mapServerBaseURL = 'https://ccis.environment.gov.za/map'
}
  ```

  - in your cli, run the command `npm run build-prod`
  - there will now be a `dist` folder in the root directory of your client
  - copy the contents of `dist` into the client folder allocated on your server 
  (in this case it's `C:\inetpub\wwwroot\NCCRD\Client`) 

  Now, if you point your browser to your app (`your/dns/nccrd/`), you should be up and running!

  To add the NDAO and the CSA, follow the same steps you did to setup the NCCRD (both for the client and API instances).