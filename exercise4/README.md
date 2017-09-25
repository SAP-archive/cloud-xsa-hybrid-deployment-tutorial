### CPL166
# Exercise 4: Adding Authentication and Authorization to your Application

Up to now we have allowed _anonymous_ access for our application. We will now introduce authentication and authorization to secure our application.  
  
**Authentication** forces users to log on and identify themselves, before they can use the application.  
**Authorization** defines what a certain user can do.  
  
Authentication and Authorization are managed using the XS Advanced UAA service which uses OAuth and supports HANA and SAML2 authentication.

<br>
<hr>
<br>

# Authentication

## Step 1 - Enable authentication for the backend module

Open the file `cpl166js/server.js` and comment the following line to _disable_ anonymous access:

```
    var options = xsjs.extend({

    //   anonymous: true, // remove to authenticate calls

       redirectUrl: “/index.xsjs”

    });
```
<br>


## Step 2 - Enable authentication for the frontend module

Configure the cpl166ui module to use authentication by opening the file `cpl166ui/xs-app.json` and changing the `authenticationMethod` attribute to `route` like this:

```
"authenticationMethod": "route",
```
<br>


## Step 3 - Protecting the frontend against Cross-Side Request Forgery (CSRF) attacks

Once we switch on the authentication for the ui module, the XS Advanced runtime will require any HTTP update request (PUT, POST, DELETE) to present a valid CSRF token. This is a standard security technique to prevent cross-side request forgery.  
The following code will fetch the CSRF token when the application is first loaded and add it automatically to the HTTP header of each update request.   
Append the following code to the end of `cpl166ui/resources/Util.js`:  

```
$(function() {
	$.ajax({
		type: 'GET',
		url: '/',
		headers: {'X-Csrf-Token': 'Fetch'},
		success: function(res, status, xhr) {
			var sHeaderCsrfToken = 'X-Csrf-Token';
			var sCsrfToken = xhr.getResponseHeader(sHeaderCsrfToken);
			$(document).ajaxSend(function(event, jqxhr, settings) {
				if(settings.type === 'POST' || settings.type === 'PUT' || settings.type === 'DELETE') {
					jqxhr.setRequestHeader(sHeaderCsrfToken, sCsrfToken);
				}
			});
		}
	});
});

```

<br>
<hr>
<br>

## Step 4 - Reference the UAA service

In order to bind our application modules to the UAA service at run time, we first need to define a new UAA service instance.
To do so, add a new resource in the mta.yaml file:

<img src="img/uaa_create.png" alt="Creation of uaa resource">

<br>

Next, we define the dependency of the `cpl166ui` and `cpl166js` modules to this resource. This will bind the modules automatically to the UAA service instance.   
Add a new entry in the requires section of both modules:  

<img src="img/js_uaa.png" alt="Reference to uaa from js module">
<br>
<img src="img/ui_uaa.png" alt="Reference to uaa from ui module">

In addition we want the authentication token acquired at runtime by the frontend (cpl166ui) module to be automatically forwarded to the back-end module (`cpl166js`).  
Add an additional property `forwardAuthToken: true` to the destinations property of the `cpl166ui` module in the `mta.yaml`:  

<img src="img/forwardAuthToken.png" alt="Add property forwardAuthToken=true">
<br>
<hr>
<br>

Continue with [Exercise5](../exercise5/README.md)
