# 4. SEB Server Setup (Web UI)

[← Back to overview](../README.md) · Previous: [3. Moodle Setup](03-moodle-setup.md)

## Add an institution

First, add an institution in the SEB Server web UI.

## Create a user

Create a user with the role **Institutional Administrator** — this user is needed to set up the LMS connection. Make sure to select the correct institution.

![Add User Account form with the Institutional Administrator role selected](images/webui-add-user-account.png)

## Add the LMS setup

Add a new LMS setup with the Moodle URL and the credentials (or access token) of the Moodle `sebserver` user created above. The user must be authorised for the web service and have the required Moodle permissions (see [3. Moodle Setup](03-moodle-setup.md)).

![Assessment Tool Setup list with the Add Assessment Tool Setup action](images/webui-assessment-tool-setups.png)

![Assessment Tool Setup detail with server address and access token](images/webui-assessment-tool-setup-detail.png)

## Exam Configuration

Create an Exam Configuration once. Alternatively, it can be imported from an existing SEB config file, so it does not have to be configured from scratch.

![Add Exam Configuration form](images/webui-add-exam-configuration.png)

Once a configuration is selected, the SEB settings can be edited (similar to the SEB client configuration tool).

![Exam Configuration detail with the Edit SEB Settings action](images/webui-exam-configuration-edit-seb-settings.png)

![SEB Settings editor](images/webui-seb-settings-browser.png)

On the *Security* tab, the allowed SEB versions can be restricted (minimum SEB client version):

![SEB Settings Security tab with Allowed SEB Versions](images/webui-seb-settings-security.png)

The configuration must be marked as **Ready To Use**, then it can be saved as a template.

## Configuration Template

Created from the Exam Configuration. Individual attributes (e.g. the clipboard policy) can be adjusted per template:

![Configuration Template attribute for the clipboard policy](images/webui-template-attribute-clipboard.png)

## Exam Template

Now create an Exam Template (selecting the Configuration Template).

![Exam Template form with Configuration Template and Connection Configuration](images/webui-exam-template.png)

Optionally, client groups (e.g. per operating system) can be added.

![Exam Template detail with the Add Client Group action](images/webui-exam-template-client-group.png)

---

Next: [5. Creating an Exam →](05-creating-an-exam.md)
