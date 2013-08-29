## The FormAssembly API

### Introduction

[FormAssembly.com](http://formassembly.com) is a web-based form management solution that allows anyone to easily create online forms and collect data.  FormAssembly is available in different flavors, cloud-hosted (SaaS) or on-premise, and all editions provide a secure REST API for interacting with user accounts and exporting data.

Before you can start building an application using the FormAssembly API, check out [the FormAssembly Developer Hub](http://www3.formassembly.com/api/) for information on how to register, obtain a CLIENT_ID and get access to a sandbox.




### Endpoint

The Endpoint, in API parlance, is the destination URL for your API requests. The Endpoint will vary depending on which FormAssembly Edition you are interacting with.

| Edition  | Endpoint | Note |
| -------- | -------- | ---- |
| FormAssembly Developer Sandbox | https://developer.formassembly.com/api_v1/ | for development and testing only. |
| FormAssembly.com | https://app.formassembly.com/api_v1/ | |
| FormAssembly Enterprise Cloud | https://instance_name.tfaforms.net/api_v1/   |  adjust instance_name as needed |
| FormAssembly Enterprise On-Site | https://your_server/path/to/formassembly/api_v1/  | adjust server and path as needed |

From this point on, we'll use the FormAssembly.com endpoint. Just remember to adjust URls to match your own FormAssembly Edition.

### Registration

Before you can interact with the API, you must obtain a CLIENT_ID and a CLIENT_SECRET code. This is done by completing the registration process on the particular instance you are targeting. For the Developer Sandbox and FormAssembly.com you must register via [the FormAssembly Developer Hub](http://www3.formassembly.com/api/). For Enterprise instances, the FormAssembly administrator can register your app.

### Authorization

The Authorization is the mechanism that lets FormAssembly users decide what application can access their account through the API. Authorizations are handled using the [OAuth2](http://oauth.net/2/) protocol.

If you're not familiar with OAuth2 interactions, it can be helpful to look at the documentation of other APIs, [like this one for the Google API](https://developers.google.com/accounts/docs/OAuth2) explaining how OAuth2 works in detail.

The key point of OAuth2 is that it allows a 'User' of FormAssembly, let's call them Adam, to authorize you (a 'Client') to take actions in FormAssembly as if you were Adam.  As an example, you could write an application which when authorized by Adam, could download and display a list of all of Adam's forms along with associated metrics, like number of responses, drop-out rate, etc.  This data could for instance be used by you to generate custom reports for Adam.

#### Step-by-Step Outline

To request a user's authorization, your application must follow these steps:

##### 1. Redirect the user to the authorization url:

https://app.formassembly.com/oauth/login?type=web&client_id=CLIENT_ID&redirect_uri=RETURN_URL&response_type=code 

where:

  + CLIENT_ID is the unique client id generated by FormAssembly for your client application (see Registration above)
  + RETURN_URL is the url your user ('Adam') will be returned to when they complete the authorization step on FormAssembly. This value must be url-encoded.  

##### 2. Retrieve the Authorization code

FormAssembly will append the 'code' query string parameter to your RETURN_URL. when returning the user to the url. For instance, a `redirect_uri=https%3A%2F%2Fwww.yourapp.com` (note the value is URL-encoded) will result in the user ('Adam') being taken to: `https://www.yourapp.com?code=XXXXXXXXXX` after authorizing FormAssembly.

Retrieve the code from the query string, you will need it to obtain the Access Token.

##### 3. Request an Access Token

Execute a HTTP request *from your server* (do *not* direct the user) to request an Access Token.

 + URL: https://app.formassembly.com/oauth/access_token
 + HTTP Method: POST
 + Post Data


 | Parameter name | value  | Comment
 | ---------- | ---------- | ----------|
 | grant_type | authorization_code | use this value as-is | 
 | type | web_server | use this value as-is |
 | client_id | CLIENT_ID | use your unique client application id issued by FormAssembly |
 | client_secret | CLIENT_SECRET | use your unique client application secret issued by FormAssembly |
 | redirect_uri | RETURN_URL | the same RETURN_URL used in step 1. |
 | code | CODE | the code parameter you obtained in step 2. |


The response to this request will be a JSON array like so:
 
    { "access_token":"XXXXXXXXXXXXXX",
      "expires_in":XXXXX,
      "scope":XXXX,
      "refresh_token":"XXXXXXXXXXXX" }

Store the access_token value. You will need it for every API requests on the user ( 'Adam' ) account. Since the access token is valid for only one user account, you will typically obtain multiple access_token from multiple users, and you must store and manage those tokens accordingly.



### Example

####  Request

A HTTP GET request to: 

https://app.formassembly.com/api_v1/forms/index.json?access_token=ACCESS_TOKEN

The URL is composed of the following parts:

 | https://app.formassembly.com/ | The FormAssembly instance |
 | api_v1 | The API version in use. As changes are made to the API, new version can be released, under api_v2, api_v3, etc. |
 | forms/index | The requested data (see API Reference below) |
 | json | The requested format for the response |
 | ACCESS_TOKEN | The token received in step 3. of the Authorization process |


#### Response 

The response from the request above will result in a JSON formatted list of the forms in the user ('Adam')'s account:

    {"Forms":[{
    "Form":{
    "id":"XXXX", "version_id":"XXXX", "name":"XXXXXXXX",
    "category":"XXXX", "subcategory":"XXXX", "is_template":"XXXX",
    "display_status":"XXXX", "moderation_status":"XXXX", "expired":"XXXX",
    "use_ssl":"XXXX", "user_id":"XXXX",
    "created":"XXXXXXXX", "modified":"XXXXXXXX",
    "Aggregate_metadata":{
    "id":"XXXX", "response_count":"XXXX", "submitted_count":"XXXX",
    "saved_count":"XXXX", "unread_count":"XXXX", "dropout_rate":"XXXX",
    "average_completion_time":"XXXX", "is_uptodate":"XXXX"
        }}
    }]}

For an explanation of each field, please see [Object Reference](#object-reference).

Several output formats are accepted, see [End Points](#end-points) for more details.

### API Reference

#### Formats:

FormAssembly supports returning data in two main formats:

  + [json](http://en.wikipedia.org/wiki/Json): a lightweight data exchange format supported by newer languages.  Directly parsable by Javascript.
  + [xml](http://en.wikipedia.org/wiki/XML):  an industry standard data exchange format, parsable by almost all languages.

Some end-points support additional formats, including:
  + [plist](http://en.wikipedia.org/wiki/Plist): used to provide Apple consumable data for Objective-C applications.
  + [csv](http://en.wikipedia.org/wiki/Comma-separated_values): used as a standard record data exchange format.
  + [zip](http://en.wikipedia.org/wiki/ZIP_(file_format\)): a binary data container format.

#### Forms: [Returned Fields Reference]

##### Index:
+ https://app.formassembly.com/api_v1/forms/index.json
+ https://app.formassembly.com/api_v1/forms/index.xml

Returns a list of the forms in the user's account, along with associated metadata.

##### Admin Index: (Enterprise plan only)
 + https://app.formassembly.com/admin/api_v1/forms/index.json
 + https://app.formassembly.com/admin/api_v1/forms/index.xml

Returns a list of all forms in the FormAssembly instance.  Only accessible if using FormAssembly Enterprise and access_token from admin level user.

Code Examples:

 + PHP: https://github.com/veerwest/formassembly-api/blob/master/php/fa_api.php
 + Python (command line): https://github.com/veerwest/formassembly-api/blob/master/python/fa_api.py
 + Bash (command line): https://github.com/veerwest/formassembly-api/blob/master/curl/fa_api.sh
 + Salesforce: https://github.com/drewbuschhorn/sfdc-oauth-playground/tree/oauth2_drew


#### Responses: [Returned Fields Reference]

##### Export:
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.csv
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.json
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.xml
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.zip

additional parameters:
+ date_from: start date for export range
+ date_to: end date for export range
+ filter: if set to 'all', export will include both completed and incomplete responses
+ response_ids: set of comma delimited response ids to retrieve

examples:
+ https://app.formassembly.com/api_v1/responses/export/1.csv?date_form=01/01/2012&date_to=01/01/2013&filter=all
+ would retrieve all responses (including incompletes) created between January 1, 2012 and January 1, 2013.
+ https://app.formassembly.com/api_v1/responses/export/1.xml?response_ids=10,11,12
+ would retrieve only responses with id's: 10,11,12 

#### Object Reference:

##### Form:

***
"id":"XXXX"

Unique integer value identifying the form within the FormAssembly instance.  Every form has a single unique id in the form of an integer.  Can be used to construct a valid form url: http://app.formassembly.com/forms/view/XXXX

Example value: 1 

***
"version_id":"XXXX"

Unique integer id identifying the current version ( revision ) of the form.

Example value: 1

***
"name":"XXXXXXXX"

HTML encoded string representing the form's name as displayed in the user's FormAssembly form index list.  Not to be confused with the Form Title, found in the form's XML definition.

Example: "Mine &amp; Yours Form"

***
"category":"XXXX",

String representing one of the system wide default form organizational categories.  Can be an empty string for uncategorized forms. 

Example: "Contact Forms"

***
"subcategory":"XXXX",

String representing one of the user created organizational categories.  Can be an empty string for uncategorized forms.

Example: "IBM Contact Forms"

***
"is_template":"XXXX",

Integer specifying whether or not the form is shared as a template publicly.

Values:

 + 0  – Not a template
 + >1 – Is a template (Contact Support for more details)

Example: 0 

***
"display_status":"XXXX",

Integer specifying whether or not the form is active or archived. Values:

 + 0 – Archived
 + 2 – Active

Example: 2

***
"moderation_status":"XXXX",

Integer specifying whether or not the form is moderated (under review for suspicious content). Values:

  + 0 – Not checked
  + 2 – Reviewed and approved
  + 3 - Reviewed and denied

Example: 0

***
"use_ssl":"XXXX",

Integer representing if the form must be displayed over HTTPS.  If true, form URL must contain https:// .

***
 + "created":"XXXXXXXX"
 + "modified":"XXXXXXXX"
 + "expired":"XXXX"

String timestamps representing the date the form was created, the last time it or any of its settings were modified, and the date it was marked for deletion if any.  

Examples:

 + "created":"1983-01-01 23:59:59"
 + "modified":"2012-06-17 17:50:12"
 + "expired":null

***

### Self-register your app on an existing Enterprise instance

If you are developing for an existing Enterprise instance you will need to self-register your app by following the instructions below. 

1. Login to your Enterprise account as an admin
2. Navigate to the **Admin** tab ![Step1](http://www3.formassembly.com/blog/wp-content/uploads/2013/08/step1.png) 
3. On the Application Settings page, navigate to the **User Settings** tab ![Step2](http://www3.formassembly.com/blog/wp-content/uploads/2013/08/step2.png)
4. Click "Register a new application" ![Step3](http://www3.formassembly.com/blog/wp-content/uploads/2013/08/step3.png)
5. Pick a name for your new application and take note of your OAuth credentials ![Step4](http://www3.formassembly.com/blog/wp-content/uploads/2013/08/step4.png)
