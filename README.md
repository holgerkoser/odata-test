[OData](http://www.odata.org/) acceptance tests with [QUnit](http://qunitjs.com/) 
===========

This project provides a [QUnit](http://qunitjs.com/) addon `qunit-odata` which helps you to 
quickly create acceptance tests for [OData](http://www.odata.org/) services.
It uses [datajs](http://datajs.codeplex.com) as OData client library. 

* The service `$metadata` are automatically read when the addon is load 
before the the test suite is started. The response data contain 
javascript Date objects instead of their String representation for Edm.DateTime
and Edm.DateTimeOffset.

* It wraps the `asyncTest` method of qunit with the new `odataTest` method and 
provides a unique callback for the success and error case.  It is not neccessary
to call start() in the odataTest callback.

* A new assertion `expectedHeaders` for http header comparison is provided.
 
* The CSRF-Protection mechanism of SAP Netweaver Gateway is automatically supported within 
this addon. This makes it simple to write test for modifying request. 

#### A minimal qunit-odata test setup
Because of the [Same-Origin Policy](http://www.w3.org/Security/wiki/Same_Origin_Policy)
this html file must be retrieved from same-origin as the odata service resources.

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <meta http-equiv='X-UA-Compatible' content='IE=edge' />
  <title>OData Test Suite</title>
  <link rel="stylesheet" href="http://code.jquery.com/qunit/qunit-git.css">
</head>
<body>
  <div id="qunit"></div>
  <script src="datajs-1.0.3.js"></script>
  <script src="http://code.jquery.com/qunit/qunit-git.js"></script>
  <script src="qunit-odata.js" data-service-root="/OData/OData.svc/"></script>  
  <script src="odata-test.js"></script>    
</body>
</html>
```

#### The contents of odata-test.js
This is an example for an odata testsuite file. The content of this 
javascript file can be retrieved from foreign origins.

```javascript
/* First Test */        
var request = {
  resourcePath: ".", 
  headers: {DataServiceVersion: "999.0"}
};
odataTest("Read with invalid DataServiceVersion", 1, request, function (response, data) {
  equal(response.statusCode, 400, "StatusCode: 400");
});

module( "Products" );    

/* Second Test */
request = { resourcePath: "Products(1)" };
odataTest("Read Entity - Product 1", 4, request , function (response, data) {
  equal(response.statusCode, 200, "StatusCode: 200");
  expectedHeaders(response.headers, { DataServiceVersion: "2.0" }, "DataServiceVersion: 2.0");
  equal(data.Name, 'Milk', "Name: 'Milk'");
  deepEqual(data.ReleaseDate, new Date("1 Oct 1995 GMT"), "ReleaseDate: 1995-10-01");
});
```
