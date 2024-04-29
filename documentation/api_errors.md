# OJP 2.0 API Errors Documentation

This document provides detailed descriptions of the error types returned by the OJP 2.0 XML API.

## XML Body Used

Check [./request-samples](./request-samples/) for request XMLs used.

Example: [./request-samples/lir-valid-request.xml](./request-samples/lir-valid-request.xml)

``` xml
<OJP xmlns:siri="http://www.siri.org.uk/siri" version="2.0"
  xmlns="http://www.vdv.de/ojp"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <OJPRequest>
    <siri:ServiceRequest>
      <siri:RequestTimestamp>2024-03-14T17:35:16Z</siri:RequestTimestamp>
      <siri:RequestorRef>OJP_Demo_iOS_0.9.1</siri:RequestorRef>
      <OJPLocationInformationRequest>
        <siri:RequestTimestamp>2024-03-14T17:35:16Z</siri:RequestTimestamp>
        <InitialInput>
          <Name>Bern</Name>
        </InitialInput>
        <Restrictions>
          <Type>stop</Type>
          <NumberOfResults>10</NumberOfResults>
        </Restrictions>
      </OJPLocationInformationRequest>
    </siri:ServiceRequest>
  </OJPRequest>
</OJP>
```

## Error Types

### 1. HTTP Errors

#### 1.1 Invalid Authorization

``` sh
curl -X POST https://api.opentransportdata.swiss/ojp20 \
  -H 'Authorization: Bearer invalid_token' \
  -H 'Content-Type: application/xml'
```

Response

```
Content-Type: application/json
Vary: Origin
X-Generator: tyk.io
Date: Thu, 18 Apr 2024 13:32:01 GMT
Content-Length: 57
Strict-Transport-Security: max-age=21600000

{
    "error": "Access to this API has been disallowed"
}
```

**TODO** - check if we can get a XML response instead

#### 1.2 Invalid Endpoint

```sh
$ curl -X POST https://api.opentransportdata.swiss/ojp20-foo \
  -H 'Authorization: Bearer ey...'

# or 

$ curl -X GET https://api.opentransportdata.swiss/ojp20-foo
```

Response

```
Content-Type: application/json
Vary: Origin
X-Generator: tyk.io
Date: Thu, 18 Apr 2024 13:32:01 GMT
Content-Length: 57
Strict-Transport-Security: max-age=21600000

<html>

<head>
    <title>404 Not Found</title>
</head>

<body>
    <center>
        <h1>404 Not Found</h1>
    </center>
    <hr>
    <center>nginx/1.20.2</center>
</body>

</html>
```

**TODO** - check if we can get a XML response instead

-> last info: response bodys for errors from the API Gateway (tyk.io) will always be returned as JSON. We can rely on HTTP Status codes for handling, as OJP domain errors will be returned as HTTP Status Code `200`

### 2. OJP Domain specific errors

#### 2.1 Invalid Requext XML

[./request-samples/lir-invalid-xml.xml](./request-samples/lir-invalid-xml.xml)

``` sh
$ curl -X GET https://api.opentransportdata.swiss/ojp20 \
  -d @documentation/request-samples/lir-invalid-xml.xml \
  -H 'Content-Type: application/xml'
```

Response

``` json
{
    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "traceId": "00-f1b8b36ecb7908e060a88be3d915bf19-2eaa00a2dc1555c2-00",
    "errors": {
        "": [
            "An error occurred while deserializing input data."
        ]
    }
}
```

## Current issues

To be discussed with the API provider

### 1. Mixed XML / JSON

OJP 2.0 XML returns a mixture of XML / JSON content for errors, would be good if this can be normalized

### 2. Unexpected error when using optional nodes

Following XML doesnt contain `<Restrictions>` node, which is optional according to [OJPLocationInformationRequestStructure](https://vdvde.github.io/OJP/develop/index.html#OJPLocationInformationRequestStructure) XSD schema

[./request-samples/lir-error-no-restriction.xml](./request-samples/lir-error-no-restriction.xml)

``` sh
$ curl -X GET https://api.opentransportdata.swiss/ojp20 \
  -d @documentation/request-samples/lir-error-no-restriction.xml \
  -H 'Content-Type: application/xml'
```

Response

``` json
{
    "traceId": "6883eadb-d859-4122-9575-b7ba6aadd3ce",
    "error": "Object reference not set to an instance of an object.",
    "host": "ODP-EC2-SHLNX02-P"
}
```
