# Cloud Elements Homework

Below are two examples of customer issues. Please research and provide steps you would take in coming up with a resolution to these prompts. Bonus points if you are able to identify the root cause or solution.
___
1.  We are trying to find a way to query for customers that have been deleted from Quickbooks Online since a certain date. For example, how can I find all customers that a user has deleted since 04-09-2019?

In recognizing that there could be a log of API requests, I did a quick search using the term ‘log’ within the Cloud Elements documentation. It pointed me right to the Audit Logs API ( https://docs.cloud-elements.com/home/audit-logs#list-api-request-logs ).

Using the GET / audit-logs, the client would be able to enter their parameters and get a response with all of the customers a specific user had deleted from April 9, 2019. An Example Request to the Audit Logs API would look like:
https://{{ceEnvironment}}.cloud-elements.com/elements/api-v2/audit-logs/common-resources?from=2019-04-19&to=2020-07-10&userId={{userId}}&nextPage={{nextPage}}&method=DELETE

The client would have to enter the {{userIdD}}, {{nexPage}}, and their specific {{ceEnvironment}} variables.

___

2. When I make a call against my Shopify instance to GET /customers with a pageSize value of 2000 it fails with a 400 response status and a message "Invalid search query provided." Why did my request fail?

In this case, I also searched the Cloud Elements Documentation and found the Pagination resource under the Platform Reference section.  In the docs, I found that in the Paginator version 3.0, there is a limit to the pageSize entered, and the client's value of 2000 is above that limit.  Using cursor pagination, the client would be to provide a ``nextPageToken`` to request the next page, and be able to get their desired ``pageSize``.  I did not know about Cursor based pagination, so I read a couple of articles to get a better understanding of it, and verify this would solve their error.

  A sample response to the client would be:

There are two types of pagination: ``page`` & ``pageSize``. ``page`` is the particular page of the results you’d like to look at, and ``pageSize`` is the number of records that come up in that page. There is also Cursor pagination, where you call ``GET /accounts `` and it returns a ``nextPageToken``.

It is not always possible to use page/pageSize across the Cloud Elements platform, but every element supports cursor based pagination.  You can maintain consistency across your integrations by using cursor based pagination for every element.

By default, the maximum pageSize is set to 1000 or the maximum vendor pageSize. The Cloud Elements Paginator 3.0 will throw an error if pageSize entered > vendor max pageSize. You can either try to scale down your pageSize value or use the Cursor pagination method described here: https://docs.cloud-elements.com/home/standardized-pagination.


Another possible reason for the "Invalid search query provided." could be that a ``page`` command was not given, only the ``pageSize`` was entered.

If client was unaware of cursor based pagination, and this was infact the issue, I could take the opportunity to go into more detail about the use cases and reasoning behind it. (If the data being requested is frequently changed, records could be redundant if querying based on specific page number.  Cursor based pagination ensures records are not redundant.)

___

3. Build an Element to any weather API that is free and can provide historical weather data. You do ​NOT​ need to add every endpoint. The only endpoints you need will be based on the use case described below.

Skipped this one due to account priveleges and an internal bug with the Cloud Elements platform.
___

4. Build a Formula - Use Case: when the formula is triggered (manually triggered formula) the user should receive a message stating the mean temperature over the previous 2 weeks (use the element you built in #3 above). The message can be an email or text message. You decide! Most messaging services have free developer options. Optional:​ Instead of messaging the mean temperature POST a CSV file to a documents hub element with the data.

Within the Cloud Elements Ecosystem, I created a formula called "Integrate with SG", there are currently 6 steps.

1. (trigger) the event that triggers the formula (it is currently manually triggerd).
2. (buildHeader) A JS script step which builds the header for next step (HTTP Request to GET weather data from the NCDC's RESTful API).
3.  (getWeatherData) HTTP GET request as previously mentioned, including the header data created in step two (${steps.buildHeader}).
4.  (formatData) The result from the HTTP Request is formated in a JS Script which takes the GET request's response, and calculates the mean temperature over a two week period, returning that number to the successive step.
5. (buildEmailBody) This fifth step is another JS Script step which takes the mean temperature, and drafts the body of the email. The JSON format of the POST request body taken from the SendGrid Element Instance's API Docs, not from the SendGrid's POST request body JSON format.

6. (sendGrid) Incorporating an Element API Request (/messages) to the SendGrid Element Instance, include the body and created in previous step as a variable ``${steps.buildEmailBody}``

