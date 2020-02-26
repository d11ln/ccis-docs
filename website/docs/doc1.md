---
id: doc1
title: Start here
sidebar_label: start
---



## Welcome!

Welcome to the Climate Change Information System (CCIS). <br />
The CCIS is a first step towards the development of an open source, standards-based, and integrated portfolio of systems that aims to eliminate duplication of effort, limits multiplication of data sources, and is re-usable on many levels of government in South Africa. 

This document aims to provide a cohesive guide to creating your own instance of the CCIS with its coupled systems (CCRD, DAO - more on that later) and dependencies. For reference, the national, maintained system can be found [here.](https://ccis.environment.gov.za). 



## Environment

If your machine does not already have the following installations, please proceed to install: 

[Node.js](https://nodejs.org/en/download/) <br />
[npm](https://www.npmjs.com/) <br />
[ReactJS](https://reactjs.org/) <br />
[SQL_Express_2017](https://www.microsoft.com/en-us/download/details.aspx?id=55994) <br />
[IIS](https://www.microsoft.com/en-za/download/details.aspx?id=2299) <br />
[Visual_Studio](https://visualstudio.microsoft.com/) or [CMake](https://cmake.org/)

## Server Requirements

Database: 
[SQL_Server_2008_R2](https://www.microsoft.com/en-za/download/details.aspx?id=22985) (or later) <br />

Size: up to 10GB per database, times 10 contributing systems - allow for 100GB relational data <br />

[IIS_v7](https://www.microsoft.com/en-za/download/details.aspx?id=2299) (or higher) <br />

Storage size: 10GB - 30GB <br /> 

[.Net_Core_Hosting_Bundle_v2.2.x](https://dotnet.microsoft.com/download/thank-you/dotnet-runtime-2.2.2-windows-hosting-bundle-installer)


## Server 

### Server Specs

Allocate a Windows server with the following specifications: <br />

```
Windows Server 2016 
Processor: Intel(R) Xeon(R) CPU E5-2630 v2 @ 2.60GHz (4 processors) 
Installed memory (RAM): 16.00GB minimum, more preferred 
System type: 64-bit Operating System, x64-based processor 
Disk Storage: 500GB will be adequate, 1TB preferred 
```

### Setting up the server

Download and install [SQL_Express_2017](https://www.microsoft.com/en-us/download/details.aspx?id=55994) <br />
Download and install [SSMS_18.4](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15#download-ssms)

#### Enable Internet Information Services (IIS)

Navigate to Server Roles and ensure that Web Server (IIS) and all its dropdowns are selected, then press 'next' <br />

In the following window, under Features, select ISS hostable web core as well as .Net Framework (and all dropdowns) <br />

After that you will land on [Security] settings, select all and do the same for IIS Management Tools and then proceed with installation. 

When prompted to do so, restart the server. 

#### Create your API foder

Navigate to `C:/inetpub/wwwroot/` and make a folder to house your API (we'll call it NCCRD) so that you have a path: `C:/inetpub/wwwroot/NCCRD` <br />

In `C:/inetpub/wwwroot/NCCRD/` make two folders, `Client` and `API`

#### Configure IIS

Open IIS and on the left-hand pane, select your server instance -> sites and delete the default website <br />

Right click on sites and select `add website` <br />

Enter the name of your web app (for the purposes of this walkthrough we'll make it `NCCRD`) and select `default app pool` <br />

Set the physical path to that which you created in the previous step, in this case it will be `C:/inetpub/wwwroot/Client` <br />

Make sure that `start website immediately` is checked

#### Connect your instance

Open SSMS and in the object explorer pane, rightclick on your server and select properties <br />

Allow SQL and Windows Auth <br />

Ensure that `Allow remote connections` is checked <br />

Continue to restarting your server.


## Database

### Restore backup

//TODO we currently only have instructions for building from backup - need to write docs for building DB from scratch

In SQL Server Manager, in the object explorer pane: rightclick on Databases and select `Restore Database...` <br />

Under `Source`, select `Device` and name your DB (in this case we'll call it `NCCRD`) <br />

Under `Destination`, enter the path to your .bak file and click `OK` <br />

To make sure the DB creation was a success, check for data in the DB folder (ie by running a query) <br />

#### Create a new login for your DB

Select username, password and set SQL Auth <br />

Under the option `Map to credentials`, select all server roles <br />

Under `User mapping`, select `YOUR INSTANCE (in this case NCCRD)` select all database permissions <br />

Under `Status`, ensure that `grant and enable login` is selected and run <br />

#### Connect DB in IIS

In IIS, in the Object Explorer pane, navigate to -> `YOUR_INSTANCE` -> `Sites`, rightclick on `YOUR_WEBAPP (in this case NCCRD)` and select `Add Application...` <br />

Set your Alias to `API` and the physical path to the API destination folder (in our case it's `C:/inetpub/wwwroot/NCCRD/API`) <br />

### Build your API

For the purpose of this walkthrough we'll be using Visual Studio, you're welcome to use other tools. 

This walkthrough assumes that you are building from the latest release (in this case the [NCCRD_REPO](https://github.com/SAEONData/NCCRD/releases)) <br />

Download and extract the .zip file and open the solution in Visual Studio <br />

In the API root folder, create a secrets.json file that contains your connection string for SQL, it will look something like this: 

```
{
  "ConnectionStrings": {
    "NCCRD": "Data source=.; initial catalog=NCCRD; user id=YOUR_USER_ID; password=YOUR_USER_PASSWORD;"
  },
  "VmsApiBaseUrl": "path/to/VMS/API"
}
```
Build the solution (rightclick on the sln file and run `Build` or pres ctrl + shift + B) <br />

In the solution explorer, rightclick on the project (NCCRD) and click `Publish` <br />

You will now be presented with the Publish wizard where you should set the following configs: <br />

Set defaults to `folder profile` and the `target location` to a folder you can easily locate for copying from <br />

In the next window, set the configuration to `release` and set dev mode to whichever you prefer <br />

Save your settings and publish! <br />

Assuming that everything went smoothly, you will have the API files populated inside the folder you published to, copy all its contents and paste it into the folder on your server that you had previously allocated for your API (in our case it's `C:/inetpub/wwwroot/NCCRD/API`) <br />





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

