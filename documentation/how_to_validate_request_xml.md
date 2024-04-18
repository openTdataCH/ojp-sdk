# HOWTO Validate OJP Request XML

There are several tools to perform XML validation against XSD schema, this document provides the approach using [xmllint](https://gnome.pages.gitlab.gnome.org/libxml2/xmllint.html) CLI on a UNIX / Mac OS system.

## Steps

**1. Use a OJP 2.0 Request XML**

- use one from https://opentransportdata.swiss/de/cookbook/ojp2entwicklung/
- or following XML
```
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
- save it on the local disk, i.e. `~/Downloads/validate-xml/ojp-lir-request.xml`

**2. Checkout XSD Schema**
- checkout the [VDVde/OJP](https://github.com/VDVde/OJP) repo from Github
```
# cd into ~/Downloads/validate-xml
$ git clone https://github.com/VDVde/OJP
# ~/Downloads/validate-xml/OJP is created
```

- for OJP version 2.0 one need to checkout `develop` branch
```
$ cd ~/Downloads/validate-xml/OJP
$ git checkout develop
```

**3. Run validation**
```
$ cd ~/Downloads/validate-xml
$ xmllint --noout --nowarning --schema OJP/OJP.xsd ojp-lir-request.xml 
```

**3a. Success response**
```
ojp-lir-request.xml validates
```

**3b. Error response**

`<InitialInput>` node expected
```
ojp-lir-request.xml:13: element Restrictions: Schemas validity error : Element '{http://www.vdv.de/ojp}Restrictions': This element is not expected. Expected is one of ( {http://www.siri.org.uk/siri}MessageIdentifier, {http://www.vdv.de/ojp}DataFrameRef, {http://www.vdv.de/ojp}Extension, {http://www.vdv.de/ojp}InitialInput, {http://www.vdv.de/ojp}PlaceRef ).

ojp-lir-request.xml fails to validate
```

Invalid type for `<RequestTimestamp>`
```
ojp-lir-request.xml:6: element RequestTimestamp: Schemas validity error : Element '{http://www.siri.org.uk/siri}RequestTimestamp': 'false' is not a valid value of the atomic type 'xs:dateTime'.

ojp-lir-request.xml fails to validate
```