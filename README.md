# RECON API
##INTRO

###What is ReCon?

<a href="http://recon.meddle.mobi/">ReCon</a> is a project run by Northeastern University intended to detect Personal Information leaks in Mobile Applications. It consists in a Machine Learning system that analyses the traffic that goes between mobile devices and the network and detects potential information that is leaked. That information is aggregated so that is possible to offer information about the insights discovered thanks to the analysis of all the PI leaks discovered in the user base of the tool. So far we have built an API on top of that aggregated data that let developers get information about which apps are leaking information (and which type of information) and which domains are receiving it.

ReCon is one of the projects that received one of the <a href="http://www.datatransparencylab.org/grantees.html"> 6 Data Transparency Lab Awards in 2015</a>. You can find more information about ReCon at their Website: http://datatransparencylab.github.io/recon/index.html.

###How is the API built

This is API is built using Firebase, this means that all the data is available in a Firebase tree structure and can be queried by either requesting the top-level branches (to get the whole set of information) or sub-branches to get following the tree branches. The responses are always JSON files that describe the tree structure corresponding to the URL that has been queried (e.g. the endpoint that has been requested)

####Firebase

Firebase is a service that let developers store information in tree structure that can be easily retrieved by developers, especially in Web environments, as the whole structure or sub-structures can be retrieved as json objects that can be easily manipulated. This means that by storing all the information in Firebase we get automatically an API to fetch data from it.

####Firebase SDK

You can query data hosted in the Firebase either by directly requesting the endpoint URLs as explained below or using any of the <a href="https://www.firebase.com/docs/"> Firebase SDKs: </a> Web, iOS, Android or REST.

####The root endpoint

The information is stored in a tree, that can be visualized at the Root URL of the Firebase. The URL of the Recon endpoint is: https://recon.firebaseio.com/.

Sub-URLs can be also used to show only part of the tree in a browser (e.g. https://recon.firebaseio.com/apps) or to get directly the JSON representation of that part of the tree (e.g. https://recon.firebaseio.com/apps.json)

####Return format

The API will return the structure of the tree serialized as a JSON object. 

####Passing Parameters

Additional parameters can be passed by adding them to the root URL endpoint

Some examples:

* Getting the whole list of applications

  In this case we just need to append the URL of the branch that contains all the apps (it's important to use the .json prefix to get the json representation and not the graphical one):
  
  https://recon.firebaseio.com/apps.json
  
* Getting the list of applications for android

  In this case we also append the android string to determine that we only want to receive the tree structure that holds information about android applications:

  https://recon.firebaseio.com/apps/android.json

* Getting the information about a particular application

  If we just want to know the information about one app, what we can do is just using the endpoint where the information of that application is stored:
  
  https://recon.firebaseio.com/apps/android/Clean%20Master.json
  
##RECON API SPECIFICATION
  
**Show Application Leakiness**
----
  This method fetches the leakiness information about the application or list of applications that match the selected criteria.

* **URL**

 - apps.json   (To retrieve the information about all the apps)
 - apps/<_platform_>.json (To retrieve the information about all the apps for a given platform)
 - apps/<_platform_>/<_appname_>.json (To retrieve the information about a particular app for a specific platform)

  Where <_platform_> could be either <i>android</i>, <i>windows</i>, <i>ios</i> and <_appname_> is the application name as registered in the Apple Store, Google Play, Windows Marketplace

* **Method:**
  
  GET

* **Success Response:**
  
  * **Code:** 200
  * **Content:** `{"nonTrackerCategories":{"ADVERTISERID":{"-KGnpWg-X95iuTDmlTfE":"gameanalytics.com","-KGnpWg-X95iuTDmlTfF":"d37gvrvc0wt4s1.cloudfront.net","-KGnpWg-X95iuTDmlTfG":"fyber.com"}},"nonTrackerCount":3,"nonTrackers":"true","popularity":61,"score":300,"trackerCategories":{"ADVERTISERID":{"-KGnpWfzHntgrvX0Klsj":"applovin.com","-KGnpWg-X95iuTDmlTfD":"a.applovin.com","-KGnpWg-X95iuTDmlTfH":"sponsorpay.com"}},"trackerCount":3,"trackers":"true"}`
  
  The content is JSON that shows the information for all the apps that match the category. For every app the following information is returned:

   * *popularity*: Popularity of the App in the corresponding Application Store
   * *score*: Leakiness score of the app. TBD: How is it calculated.
   * *trackers*: Whether the app is sending Personal Information to domains categorised as trackers or not.
   * *trackerCategories*: An array that contains all the information categories that are leaked to tracker domains. For every category, the list of receiving domains are offered, for instance: <code>{"trackerCategories":{"AdvertiserID":{"url1":"adkmob.com"},"AndroidID":{"url1":"adkmob.com"}}</code>
   * *nonTrackers*: Whether the app is sending Personal Information to domains categorised as trackers or not.
   * *nonTrackerCategories*: An array that contains all the information categories that are leaked to non tracker domains. For every category, the list of receiving domains are offered, for instance: <code>{"nonTrackerCategories":{"AndroidID":{"url1":"ksmobile.com"}}</code>
   
* **Error Response:**

  If no data is found for that application, the API will return a null value as reponse.

* **Sample Call:**

  * **Pure JavaScript via JSONP - NO SDK**

```javascript
<head>
  <script>
    function gotData(data) {
      console.log(JSON.stringify(data)); // contains the JSON result
    }
    function retrieveReconData() {
      var scriptE = document.createElement('script');
      // Set the source of the script element to the JSONP endpoint
      scriptE.src = 'https://recon.firebaseio.com/apps/android/Bingo%20Pop.json?callback=gotData';
      // append the script element to the page <head>
      document.getElementsByTagName('head')[0].appendChild(scriptE);
    }
  </script>
</head>
<body onload="retrieveReconData()">
</body>
```
 
  * **JavaScript using Firebase Web SDK**

```javascript
<head>
  <script src="https://cdn.firebase.com/js/client/2.4.2/firebase.js"></script>
    <script>
      var myFirebaseRef = new Firebase("https://recon.firebaseio.com/apps/android/Bingo%20Pop");

      function retrieveReconData() {
        myFirebaseRef.once("value", function(snap) {
          console.log("read data!");
          console.log(JSON.stringify(snap.val()));
        });
      }
  </script>
</head>
<body onload="retrieveReconData()">
</body>
```
* **Complete working examples:**

Complete fully functional working examples can be found at:

- http://datatransparencylab.github.io/recon-api/getAppInfoJS.html
- http://datatransparencylab.github.io/recon-api/getAppInfoFirebase.html

**Show Domain Leakiness**
----
  This method fetches the leakiness information about a particular domain.

* **URL**

 - domains/<_domainurl_>.json   (To avoid issues with encoding, the domainurl must be encoded in base16)
 
 For instance: 
 
 https://recon.firebaseio.com/domains/61646b6d6f622e636f6d.json
 
 (where 61646b6d6f622e636f6d is the domain adkmob.com encoded in base16.

* **Method:**
  
  GET

* **Success Response:**
  
  * **Code:** 200
  * **Content:** `{"apps":{"android":{"Coin Dozer - Free Prizes!":{"-KGnpWjQjChcbv8115jo":"IMEI","-KGnpWjRYr0l6wuGOjNS":"ANDROIDID"},"Prize Claw 2":{"-KGnpWjQjChcbv8115jk":"ADVERTISERID","-KGnpWjQjChcbv8115jq":"IMEI","-KGnpWjRYr0l6wuGOjNU":"ANDROIDID"},"TropWorld Casino":{"-KGnpWjQjChcbv8115ji":"ADVERTISERID","-KGnpWjQjChcbv8115jm":"IMEI","-KGnpWjQjChcbv8115js":"GENDER","-KGnpWjQjChcbv8115ju":"ANDROIDID"}},"ios":{"1010!":{"-KGnpWjRYr0l6wuGOjNW":"IDFA"},"Criminal Case":{"-KGnpWjRYr0l6wuGOjNY":"IDFA"}}},"categories":{"ADVERTISERID":{"android":{"-KGnpWjQjChcbv8115jj":"TropWorld Casino","-KGnpWjQjChcbv8115jl":"Prize Claw 2"}},"ANDROIDID":{"android":{"-KGnpWjRYr0l6wuGOjNR":"TropWorld Casino","-KGnpWjRYr0l6wuGOjNT":"Coin Dozer - Free Prizes!","-KGnpWjRYr0l6wuGOjNV":"Prize Claw 2"}},"GENDER":{"android":{"-KGnpWjQjChcbv8115jt":"TropWorld Casino"}},"IDFA":{"ios":{"-KGnpWjRYr0l6wuGOjNX":"1010!","-KGnpWjRYr0l6wuGOjNZ":"Criminal Case"}},"IMEI":{"android":{"-KGnpWjQjChcbv8115jn":"TropWorld Casino","-KGnpWjQjChcbv8115jp":"Coin Dozer - Free Prizes!","-KGnpWjQjChcbv8115jr":"Prize Claw 2"}}},"tracker":false,"url":"kontagent.net"}`
  
  The content is JSON that shows the information for the Personal Information the domain receives from mobile apps:

   * *tracker*: Whether the domain has been categorised as traker or not
   * *url*: Real URL of the domain.
   * *categories*: An array that includes all the list of categories of Personal Information that the domain receives. For every category, the list of apps doing so it is also included, e.g.: <code>"categories":{"AdvertiserID":{"app1":"Clean Master"},"AndroidID":{"app1":"Clean Master"}}</code>
   * *apps*: An array that contains all the information about the apps that leak information to that domain. For every application, the list of types of Personal Information is included, for instance: <code>"apps":{"android":{"Clean Master":{"category1":"AndroidID","category2":"AdvertiserID"}}}</code>
 
* **Error Response:**

  If no data is found for that domain, the API will return a null value as reponse.

* **Sample Call:**

  * **Pure JavaScript via JSONP - NO SDK**
  
  A full example is available at: http://datatransparencylab.github.io/recon-api/getDomainInfoJS.html the key snippet is
  
```javascript
<head>
  <script>
  function gotData(data) {
    console.log(JSON.stringify(data)); // contains the JSON result
  }
  
  function retrieveReconData() {
    var scriptE = document.createElement('script');
    // Set the source of the script element to the JSONP endpoint
    scriptE.src = 'https://recon.firebaseio.com/domains/6b6f6e746167656e742e6e6574.json?callback=gotData';
    // append the script element to the page <head>
    document.getElementsByTagName('head')[0].appendChild(scriptE);
  }
  </script>
</head>
<body onload="retrieveReconData()">
</body>
```
  * **JavaScript using Firebase Web SDK**
```javascript
<head>
  <script src="https://cdn.firebase.com/js/client/2.4.2/firebase.js"></script>
    <script>
      var myFirebaseRef = new Firebase("https://recon.firebaseio.com/domains/6b6f6e746167656e742e6e6574");

      function retrieveReconData() {
        myFirebaseRef.once("value", function(snap) {
          console.log("read data!");
          console.log(JSON.stringify(snap.val()));
        });
      }
  </script>
</head>
<body onload="retrieveReconData()">
</body>
```

* **Complete working examples:**

Complete fully functional working examples can be found at:

- http://datatransparencylab.github.io/recon-api/getDomainInfoJS.html
- http://datatransparencylab.github.io/recon-api/getDomainInfoFirebase.html



**Show Personal Information Category Leakiness**
----
  This method fetches all the information about a Personal Information Category.

* **URL**

 - categories/<_category_id_>.json   
 
 For instance: 
 
https://recon.firebaseio.com/categories/DOB.json

* **Method:**
  
  GET

* **Success Response:**
  
  * **Code:** 200
  * **Content:** `{"apps":{"All":["Run with Map My Run(ios)","Gaana(ios)"],"ios":["Run with Map My Run","Gaana"]},"description":"Birth Date","group":"PERSONAL_INFO","nonTrackers":{"All":["www.mapmyfitness.com","gaana.com"],"ios":["www.mapmyfitness.com","gaana.com"]},"numberApps":{"All":2,"android":0,"ios":2,"windows":0},"numberNonTrackers":{"All":2,"android":0,"ios":2,"windows":0},"numberTrackers":{"All":0,"android":0,"ios":0,"windows":0}}`
  
  The content is JSON that shows the information for that particular category:

   * *description*: A description for the category ID (e.g. Birth Date)
   * *group*: The top level group of information in which the category fits (e.g. PERSONAL_INFO)
   * *apps*: An array that includes the apps aggregated for all the operating systems and detailed for each one, e.g.: <code>"apps":{"All":["Run with Map My Run(ios)","Gaana(ios)"],"ios":["Run with Map My Run","Gaana"]}</code>
   * *nonTrackers*: An array that contains all the non tracker domains that receive this type of information, aggregated and detailed  per operating system, for instance: <code>"All":["www.mapmyfitness.com","gaana.com"],"ios":["www.mapmyfitness.com","gaana.com"]},"numberApps":{"All":2,"android":0,"ios":2,"windows":0</code>
   * *trackers*: An array that contains all the tracker domains that receive this type of information, aggregated and detailed  per operating system
   * *numerApps*: An array that includes the number of apps sending information for all the operating systems and per operating system, e.g. <code>{"All":2,"android":0,"ios":2,"windows":0}</code>
   * *numerNonTrackers*: An array that includes the number of non tracker domains receiving information aggregated for all the operating systems and detailed per operating system, e.g. <code>{"All":2,"android":0,"ios":2,"windows":0}</code>
   * *numerTrackers*: An array that includes the number of  tracker domains receiving information aggregated for all the operating systems and detailed per operating system, e.g. <code>{"All":2,"android":0,"ios":2,"windows":0}</code>
 
* **Error Response:**

  If no data is found for that domain, the API will return a null value as reponse.

* **Sample Call:**

  * **Pure JavaScript via JSONP - NO SDK**
  
```javascript
<head>
  <script>
  function gotData(data) {
    console.log(JSON.stringify(data)); // contains the JSON result
  }
  
  function retrieveReconData() {
    var scriptE = document.createElement('script');
    // Set the source of the script element to the JSONP endpoint
    scriptE.src = 'https://recon.firebaseio.com/domains/DOB.json?callback=gotData';
    // append the script element to the page <head>
    document.getElementsByTagName('head')[0].appendChild(scriptE);
  }
  </script>
</head>
<body onload="retrieveReconData()">
</body>
```
  * **JavaScript using Firebase Web SDK**
```javascript
<head>
  <script src="https://cdn.firebase.com/js/client/2.4.2/firebase.js"></script>
    <script>
      var myFirebaseRef = new Firebase("https://recon.firebaseio.com/domains/DOB");

      function retrieveReconData() {
        myFirebaseRef.once("value", function(snap) {
          console.log("read data!");
          console.log(JSON.stringify(snap.val()));
        });
      }
  </script>
</head>
<body onload="retrieveReconData()">
</body>
```

* **Complete working examples:**

Complete fully functional working examples can be found at:

- http://datatransparencylab.github.io/recon-api/getCategoryInfoJS.html
- http://datatransparencylab.github.io/recon-api/getCategoryInfoFirebase.html

* **Notes:**

The list of categories used so far is:

- LOCATION
- X_WP_DEVICE_ID
- MUID
- X_WP_ANID
- LASTNAME
- ADVERTISERID
- ANDROIDID
- MACADDR
- SERIALNUMBER
- FIRSTNAME
- IMEI
- GENDER
- ZIPCODE
- USERNAME
- PASSWORD
- EMAIL
- CONTACTNAME
- IDFA
- DEVICENAME
- CONTACTNUMBER
- FULLNAME
- ADDRESS
- MEID
- DOB
- PSWD
- PROFILE
- RELATIONSHIP

## USAGE IDEAS

### Interactive Visualizations

One potential usage of this API is providing an interactive visualization about how mobile applications are leaking information, which type of information and to which domains. <a href="http://datatransparencylab.github.io/recon/">A draft visualization is already available at the DTL GitHub page</a>.

### Browser Plugin 

Another potential usage is developing a Browser plugin that when browsing iTunes or Google Play pages for an application, provides information about the potential information leakages of that application.

Also, the plugin could be used to inform users when they are browsing a website that is receiving Personal Information from mobile applications.

### How much does your device leak?

Another alternative would be developing an application that inspect all the applications installed in the device and let the users know how "leaking" the device is based on the leakiness of the individual applications.

## API LIMITATIONS

The number of users of the system that generate the aggregated data that is exposed is limited so the dataset might not cover all the geographies and potentially leaking apps. Additionally, the API is not feeding the dataset in real-time yet.

## FEEDBACK

Both the project and the API are in a very preliminary status. Hence we would be grateful about any feedback you can give us!

Types of feedback we are eager to get:

- Is any missing information that could make this data more useful?
- Is there are any better way to structure the information?
- Is there any missing API?

## CONTACTING US

Join #datatransparencylab on irc.freenode.net to chat with us real time, or leave a message that will be logged 

## SOME USEFUL LINKS

### ReCon Project

http://recon.meddle.mobi/

### Data Transparency Lab

http://www.datatransparencylab.org/

### Firebase SDK

https://www.firebase.com/docs/

### Sample Visualizations

http://datatransparencylab.github.io/recon/

### Usage Examples

  - http://datatransparencylab.github.io/recon-api/getAppInfoJS.html
  - http://datatransparencylab.github.io/recon-api/getAppInfoFirebase.html
  - http://datatransparencylab.github.io/recon-api/getDomainInfoJS.html
  - http://datatransparencylab.github.io/recon-api/getDomainInfoFirebase.html
  - http://datatransparencylab.github.io/recon-api/getCategoryInfoJS.html
  - http://datatransparencylab.github.io/recon-api/getCategoryInfoFirebase.html
