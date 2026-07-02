# 3. Moodle Setup

[← Back to overview](../README.md) · Previous: [2. Docker Setup](02-docker-setup.md)

## Install the plugin

Install the following plugin, ideally via Git submodules (or manually if that is not possible):

https://github.com/ethz-let/moodle-quizaccess_sebserver

**Optional code tweak:** to make the "Quit Safe Exam Browser" button more visible, replace `btn-secondary` with `btn-primary` in `mod/quiz/accessrule/sebserver/rule.php` (around line 88).

## Create a Moodle user for the SEB Server

Create a dedicated Moodle user (e.g. `sebserver`) that the SEB Server will use to connect to Moodle.

## Web service permissions

Add the `sebserver` user as an authorised user of the plugin's external web service:

`Site administration → Server → External services` (`/admin/settings.php?section=externalservices`)

## User permissions

Create a dedicated role for the web service user. A role template is provided by the SEB Server project:

- Role template: https://github.com/SafeExamBrowser/seb-server/blob/master/docs/files/webservice_seb-server.xml
- Note: the template on the `master` branch is more complete than the one in `rel-1.5.1`

Add the role under `Site administration → Users → Define roles` (`/admin/roles/manage.php`).

Required allow capabilities for SEB Server 1.5:

```xml
<allow>mod/quiz:manage</allow>
<allow>mod/quiz:view</allow>
<allow>mod/quiz:viewreports</allow>
<allow>moodle/backup:backupcourse</allow>
<allow>moodle/category:viewcourselist</allow>
<allow>moodle/category:viewhiddencategories</allow>
<allow>moodle/course:update</allow>
<allow>moodle/course:useremail</allow>
<allow>moodle/course:view</allow>
<allow>moodle/course:viewhiddencourses</allow>
<allow>moodle/course:viewhiddensections</allow>
<allow>moodle/course:viewhiddenuserfields</allow>
<allow>moodle/course:viewparticipants</allow>
<allow>moodle/site:accessallgroups</allow>
<allow>moodle/user:update</allow>
<allow>moodle/user:viewdetails</allow>
<allow>moodle/user:viewhiddendetails</allow>
<allow>moodle/webservice:createmobiletoken</allow>
<allow>moodle/webservice:createtoken</allow>
<allow>quizaccess/seb:bypassseb</allow>
<allow>quizaccess/sebserver:managesebserver</allow>
<allow>report/courseoverview:view</allow>
<allow>webservice/rest:use</allow>
```

Then assign this role to the `sebserver` user on system level: `Site administration → Users → Assign system roles` (`/admin/roles/assign.php?contextid=1`).

## Access token

Create a web service access token for the `sebserver` user with a Moodle administrator account:

`Site administration → Server → Manage tokens` (`/admin/webservice/tokens.php`)

It is recommended to restrict the token to the IP address of the SEB Server.

---

Next: [4. SEB Server Setup (Web UI) →](04-seb-server-webui.md)
