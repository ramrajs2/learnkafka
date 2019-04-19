# Create Message Daemon Archetype

## Introduction

In this lab we'll create a message daemon application using Altus.

Altus creates an application based on the Raptor Message Daemon Archetype.
The archetype provides a template for a valid message daemon application.
We will explore this application and use it later to build our own version.

## Create Message Daemon Using Altus

### Steps

##### 1. Logon to Altus

You should already have seen how to do this, however... just in case.

    a. Open https://engineering.paypalcorp.com/altus/
    b. Logon to Altus using your corporate credentials

##### 2. Start the application creation wizard

<img src="pics/Altus_CreateApp_01.png"/>

Click on the `+ Create Application` button

##### 3. Select application stack

<img src="pics/Altus_CreateApp_02.png"/>

Select the 'Java Message Daemon' option.

When hovering the mouse over the option, the image will change to show:

<img src="pics/Altus_CreateApp_03.png"/>

Click on `Select`.

##### 4. Wait for the Creation Wizard to open

<img src="pics/Altus_CreateApp_04.png"/>

Next you'll be presented with a screen asking you for meta-data about your application.

##### 5. Fill out the application meta-data

<img src="pics/Altus_CreateApp_05.png"/>

Required fields to fill out (excluding the default values) are:
- **App name**.
  Use a name that makes sense to you.
  Notice that the field automatically fills in the required `d` at the end of the name.
  In addition to the required end character, the field will also enforce any other naming rules current for Altus.
- **Description**.
  Use a sensible description.
  This description will be used by the search engine, so for real applications you may want to put some effort into providing a good description.
- **On-call manager**.
  Find your manager in the dropdown list.
  If you can't find your manager, use `pgraff`.

Other fields to note on this screen are:
- **Messaging System**.
  We'll use the default `YAM` in this course.
  However, some of you may be exposed to another messaging system (older legacy) called AMQ (Atlas Message Queue).
- **Git Repo**.
  In the class we'll use your repo as the user.
  In a real app, you would probably use your organization's repo.
- **Owners**.
  For a real app, you would probably add a set of owners.

Finally, when you are done filling in the form, click `Next`.

##### 6. Specify outgoing connections

<img src="pics/Altus_CreateApp_06.png"/>

The next screen allows you to configure the outgoing connections from your application.

We'll stick with the defaults and simply press `Next`.

##### 7. Specify the required infrastructure

<img src="pics/Altus_CreateApp_07.png"/>

This screen allows you to configure the build pipelines and the size of the default user stage.

We'll stick with the defaults and simply press `Next`.

##### 8. Review and confirm choices

<img src="pics/Altus_CreateApp_08.png"/>

The final screen presented by the wizard is a confirmation screen that allows you to review your choices.

Let's just accept them and press `Create Application`.

##### 9. Wait for the creation of the GitHub repository

It may take a few minutes to create the application, however, as soon as the GitHub repo has been created, we should be ready to try it out locally.

## Making the application run locally

### Steps

##### 1. Clone the GitRepo

First you have to find the URL for your git Repo.
You can find the GitHub URL in Altus by looking at the App Info.

<img src="pics/Altus_AppInfo_01.png"/>

Copy the GitHub URL and in the directory of your choosing clone the repo. E.g.:

```bash
  git clone https://github.paypal.com/pgraff/pgmsgd
```
##### 2. Import the project into your IDE

The following instructions are for RIDE. If you use a different IDE, refer to that IDE's help files to find out how to import the project.
