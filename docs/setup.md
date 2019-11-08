---
id: setup
title: Start here
sidebar_label: start
---



## Welcome!

Welcome to the Climate Change Information System. 

## Environment

You'll want to make sure that your environment is setup with the following: 

NodeJS <br />
npm or yarn <br />
React <br />
SQL Server <br />
IIS <br />
Visual Studio Code (for easier builds, optional)

## Initial Setup

//TODO steps to setup db and server

## Quick note

This guide assumes usage of npm package manager, but you are more than welcome to use yarn instead if that's your preference. 

## Preflight configuration

Before you run the install and fire up your app you'll need to do some minor configuration to ensure everything is wired up for your instance

//TODO ports, secrets etc

## First run

Fork or clone this repo, cd into the client folder and run: 

```
npm i
```

You'll want to watch for any warnings in the cli, any references to missing dependencies means that you'll have to install those yourself, for example: 

```
npm WARN @ant-design/icons-react@1.1.2 requires a peer dependency of @and-design/icons but none is installed. You must install peer dependencies yourself. 
```
In the event that you come across any of these that hinder the installation or rendering of the app, simply run: `npm i` followed by the relevant dependencies as pointed out by the warnings.

If all went well and npm installed successfully, you can point your browser to the relevant port where you'll see an instance of the app. 

