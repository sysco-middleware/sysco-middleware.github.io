---
layout: post
title: Handle header parameters into split join for OSB 12c
categories: OSB tips
tags: [OSB, Spit join]
author: cliops
---
In this post, we will see a way to receive and transfer header parameters into a split join component using Jdeveloper 12c. There we go:

### Create the split join WSDL ###

First, we need to create a WSDL file for the split join component.

```xml
<?xml version= '1.0' encoding= 'UTF-8' ?>
<wsdl:definitions name="SplitJoinService" targetNamespace="http://xmlns.oracle.com/TestSplitJoin/SplitJoinService"
                  xmlns:tns="http://xmlns.oracle.com/TestSplitJoin/SplitJoinService"
                  xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
                  xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">
  <wsdl:types>
    <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified"
                targetNamespace="http://xmlns.oracle.com/TestSplitJoin/SplitJoinService">
      <xsd:complexType name="header_parameters_type">
        <xsd:sequence>
          <xsd:element name="string_parameter" type="xsd:string"/>
          <xsd:element name="int_parameter" type="xsd:int"/>
        </xsd:sequence>
      </xsd:complexType>
      <xsd:complexType name="body_type">
        <xsd:sequence>
          <xsd:element name="value" type="xsd:string"/>
        </xsd:sequence>
      </xsd:complexType>
      <xsd:element name="header_parameters" type="tns:header_parameters_type"/>
      <xsd:element name="content_body" type="tns:body_type"/>
    </xsd:schema>
  </wsdl:types>
  <wsdl:message name="requestMessage">
    <wsdl:part name="header_parameters" element="tns:header_parameters"/>
    <wsdl:part name="content_body" element="tns:content_body"/>
  </wsdl:message>
  <wsdl:portType name="execute_ptt">
    <wsdl:operation name="execute">
      <wsdl:input message="tns:requestMessage"/>
    </wsdl:operation>
  </wsdl:portType>
  <wsdl:binding name="execute_bind" type="tns:execute_ptt">
    <soap:binding transport="http://schemas.xmlsoap.org/soap/http"/>
    <wsdl:operation name="execute">
      <soap:operation style="document" soapAction="execute"/>
      <wsdl:input>
        <soap:body parts="content_body" use="literal"
                   namespace="http://xmlns.oracle.com/TestSplitJoin/SplitJoinService"/>
        <soap:header message="tns:requestMessage" part="header_parameters" use="literal"/>
      </wsdl:input>
    </wsdl:operation>
  </wsdl:binding>
</wsdl:definitions>

```
As you can appreciate in the code above, there is a header section (soap:header) at the end of the document, so it will be necessary to be used in the split join.

### Create the split join component ###

Create a split join component from the composite and use the WSDL previously created above.

![](/images/2018-02-27-split-join-transport-header/Image1.jpg)

It will look like the image below
![](/images/2018-02-27-split-join-transport-header/Image2.jpg)

Open the split join and include an assign component.
![](/images/2018-02-27-split-join-transport-header/Image3.jpg)

Then, In the image below we can find parameter headers defined in the wsdl.
![](/images/2018-02-27-split-join-transport-header/Image4.jpg)

Now, what is important if we want to transfer the header to another service then just is necessary to replace the value of request.header_parameters with some another value or xml structure that corresponds to the wsdl of the external service. Finally, add the invoke service and set the Request variable as the same request used before.
![](/images/2018-02-27-split-join-transport-header/Image5.jpg)


### Conclusion ###

This is one of the solutions to handle headers into split join that I consider as good enough. If you have any suggestion or question let me know.
