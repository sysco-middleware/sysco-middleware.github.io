---
layout: post
title: Using Postman to quickly view responses of the same endpoint with different inputs
categories: testing
tags: [testing,postman]
author: AnitaLipsky 
---

# Purpose
If you quickly wish to iterate over the same API endpoint using different inputs for that one endpoint and view the responses, this is for you.


## Example
View the different responses from a “get customer info” API to see which fields and what kind of data is returned for various types of customers.



# Pre requisites
* Postman is installed
* The URL is known along with a series of data values, eg customer ids



# Steps

## Try first with one hardcoded request/response
Create a collection in Postman and one request inside the collection with one value hardcoded, eg customerId is 483228

Save the request.

Test that one request returns the expected result and when satisfied, go to the next step.

Request example:
```
https://mydomain/myapp/api/v1/customer/483228
```


## Update the request to use a variable
Update the request by replacing the hardcoded value with a variable inside double curly brackets, eg{% raw %} {{custId}} {% endraw %}

Save the request.

Request example:
{% raw %}
https://mydomain/myapp/api/v1/customer/{{custId}}
{% endraw %}


## Create a .csv file with values for the variable
Create a .csv file with a heading that matches the variable name.

Add one row for each variable value, eg each customerId

Note that Postman accepts other data file types such as application/json though for this example text/csv is used.

Example customerId.csv contains:
```
custId
15
16
1054
1141
1211
225880
290188
483228
```


## Run the request in Runner
Open the runner - look for the light grey “Runner” button in the top left hand part of the screen which can be hard to notice.

![Postman runner button](/images/2020-03-05-Using-Postman-to-quickly-view-responses-of-the-same-endpoint-with-different-inputs/postman-runner-button.png)

Select the collection that you created containing the request to iterate over.

Select the file by clicking Data File Type in the left hand part of the screen.

Postman will run the same request for each line of data in the file, so click Preview next to Data File Type to be sure the data looks ok.

Click the big blue Run button towards the bottom of the screen. If this button is greyed out, be sure you selected a collection.


## Quickly view the responses
The default screen contains a lot of information that I find involves too much clicking and right-clicking to view the results, so I recommend opening the Postman Console.

Open this by going to the top menu bar and selecting View > Show Postman Console

This will open a new window.

Expand the request to see its response and expand its response body.

![Postman interations one response](/images/2020-03-05-Using-Postman-to-quickly-view-responses-of-the-same-endpoint-with-different-inputs/postman-iterations-one-response.png)

Scrolling down it is nice to see the arrays collapsed by default which gives a quick overview of the data.

![Postman response with arrays](/images/2020-03-05-Using-Postman-to-quickly-view-responses-of-the-same-endpoint-with-different-inputs/postman-response-with-arrays.png)

Looking through the responses for various customerIds it is easy to get a quick overview of the types of customers by viewing the values of “type” in the responses and their other values returned.

In this particular response we also see 37 addresses which indicates this customer has been used for testing since it is not a real world scenario to have 37 addresses for a customer.


## Conclusion
Using Postman Runner is a quick way to get an overview of an endpoint that is populated with some data.

It can be helpful especially during the early stages of development and testing when getting a feel for the APIs and the application in general.

