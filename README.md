JASIG CAS EXAMPLE EXTENSIONS
============================

Example JSIG CAS WAR Overlay project with various CAS extension modules

Example extensions to the standard JASIG CAS SSO Server by symentis GmbH, Robert Oschwald

This project holds the following CAS Server extensions:

WebserviceAuthenticationHandler
-------------------------------
Module cas-server-support-webservice is a sample WebserviceAuthenticationHandler implementation you can use to authenticate
against a Webservice based backend. The WebserviceAuthenticationHandler is webservice technology agnostic.
Simply wire in your WebserviceClient implementation (e.g. SOAP or REST client).
Provided is a Spring-WS based Webservice client which can be configured to run with- or without a WSSE header.

DirectMappedPersonAttributeDao
------------------------------
By default, the PersonAttributeDao implementations of the Jasig Person-Directory library need an extra request
after sucessful authentication to pull user attributes to provide them to CAS Client applications.
The DirectMappedPersonAttributeDao is a short-term caching attributeRepository, which can be filled with user attributes
from beans directly (e.g. by AuthenticationHandlers).

This example CAS Server application wires the DirectMappedPersonAttributeDao into the WebserviceAuthenticationHandler.
On successful authentication, the received principal attributes are stored in the DirectMappedPersonAttributeDao.
On first serviceValidate state, the attributes for the principal are removed from the short-term cache.
Stale entries (e.g. no serviceValidate state happened) are removed after a TTL (default 1 minute).
See the provided deployerConfigContext.xml file for an example configuration of this attributeRepository.

Example serviceValidate response:

```xml
<cas:serviceResponse xmlns:cas='http://www.yale.edu/tp/cas'>
    <cas:authenticationSuccess>
        <cas:user>admin</cas:user>
        <cas:attributes>
            <cas:isRemembered></cas:isRemembered>
            <cas:isFromNewLogin>false</cas:isFromNewLogin>
                <cas:USER_ATTRIB_CACHE_EXPIRY_TIME>1397730496188</cas:USER_ATTRIB_CACHE_EXPIRY_TIME>
                <cas:lastname>TestLastName</cas:lastname>
                <cas:netid>admin</cas:netid>
                <cas:firstname>TestFirstName</cas:firstname>
        </cas:attributes>
    </cas:authenticationSuccess>
</cas:serviceResponse>
```

To prevent the USER_ATTRIB_CACHE_EXPIRY_TIME attribute to be returned, do not select the "Ignore Attribute Management via this Tool" checkbox in the service management application.
Instead, select the attributes you want to have returned to services on a per-service base.

Project setup
-------------
After you checked out the code from the repository, it is mandatory that you perform a mvn install run.
This creates the JaxB classes for the sample SOAP webservice in target/src

Sample application
------------------
The CAS application maven overlay configuration in the cas-server-webapp module of this project uses

 * Spring-WS based WebServiceClient to authenticate against a provided test Spring-WS WebserviceEndpoint (which btw. accepts every username / password given).
   The WebserviceClient is configured to not use a WSSE header.
 * Configures the DirectMappedPersonAttributeDao to provide CAS Client applications user attributes received by the WebserviceClient.

Configuration
-------------
 * cas-server-webapp/src/main/webapp/WEB-INF/deployerConfigContext.xml
 * cas-server-webapp/src/main/webapp/WEB-INF/spring-ws-config.xml
 * cas-server-webapp/src/main/webapp/WEB-INF/spring-configuration
 * cas-server-webapp/src/main/webapp/WEB-INF/webservice-configuration
 * cas-server-webapp/src/main/webapp/WEB-INF/web.xml
   This is the original CAS Server 3.5.3.1 web.xml file plus Spring-WS MessageDispatcherServlet config at the bottom.
   Added for the test Spring-WS ExampleAuthenticationEndpoint.
 * cas-server-webapp/src/main/webapp/view/jsp/protocol/casServiceValidationSuccess.jsp (added the cas attributes to the view)


Running the sample CAS application
----------------------------------
The "integration-test" phase of this Maven project provides a quick self-contained
[JASIG CAS](http://jasig.org/cas) demo environment, performing the following:
 * Downloads the JASIG CAS project war and overlays that with the local modifications of this project.
 * Generates an SSL cert for HTTPS access, and exports the public cert in PEM format for use by CAS clients,
 * Launch CAS on an embedded Tomcat 7 instance (ports 8080 and 8443)
 * Grants access to the 'admin' account for the Services Management interface.
Thanks to the work of Matt Forsetti. See https://github.com/forsetti/jasig-cas-quickdemo

Running the application:
```
git clone https://github.com/robertoschwald/jasig-cas-examples-robertoschwald.git
cd jasig-cas-examples-robertoschwald
mvn integration-test
```

Then access https://localhost:8443/cas/ in your favorite browser.
The Services Management interface can be accessed at
 https://localhost:8443/cas/services/manage.html (log in as admin:anypass)

Warning
-------
*DO NOT USE THIS PROJECT AS PART OF ANY PRODUCTION BUILD*.
Instead, use a separate Java application server (Tomcat, JBoss, etc), properly secured,
and build a securely configured CAS server bundle to be deployed into that app server.


Defaults
--------
The following defaults can be overidden using Maven's standard property
override.  For example, to run HTTPS on TCP PORT 9443, add
"-Dhttps.port=9443" to the command line.

```
cas.version=3.5.2
http.port=8080
https.port=8443
keystore.file=quickdemo.jks
keystore.pem=quickdemo.pem
keystore.pass=quickdemo
keystore.dir=target/security
```

Keystore files
--------------
JASIG CAS and the WebserviceClient rely on HTTPS. Therefore, the maven build creates a SSL Certificate and
stores it in a Java keystore file used by the embedded Tomcat server, so the underlying Java implementation trusts this
self-signed certificate (The WebserviceClient otherwise would not accept the connection)

 * The JKS and PEM files are available in cas-server-webapp/target/security. (Useful for testing external CAS clients as well)
 * CAS server logs are in target/tomcat/logs/

CAS URLs
--------
Login: https://localhost:8443/cas/login
Services Management: https://localhost:8443/cas/services/manage.html
CAS 2.0 Service Validation: https://localhost:8443/cas/serviceValidate

More Information
----------------
 * [JASIG CAS](http://jasig.org/cas)
 * [JASIG CAS on GitHub](https://github.com/jasig/cas)


LICENSE
-------
Copyright 2014 symentis GmbH

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.



