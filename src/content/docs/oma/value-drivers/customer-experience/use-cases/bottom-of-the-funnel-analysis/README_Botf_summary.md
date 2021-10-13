# Bottom of the funnel summary data queries
Edit the title tile to reflect the bottom of the funnel process shown in the dashboard.

## Conversion Funnel 
Update the Funnel query for your bottom of the funnel process
The structure of the funnel query is as follows: <br>
>FROM DATA_TYPE <br>
  SELECT funnel(COMMON_ATTRIBUTE, WHERE .. ACTION_A, .., WHERE .. ACTION_N) <br>
  SINCE 1 WEEK AGO <br>

COMMON_ATTRIBUTE - usually set to session.  You can capture conversions that happen over multiple sessions, by adding [custom attributes](https://docs.newrelic.com/docs/browser/new-relic-browser/browser-agent-spa-api/setcustomattribute-browser-agent-api/), like customerId, to Browser.  

WHERE … ACTION - filters for the PageViews or AjaxRequests relevant to that step in the user’s journey.  

__Examples__  <br>
The following tracks conversions on the basis that a user converts in a single browser session (__PageViews__)
>FROM PageView SELECT funnel(session_id, <br>
      WHERE appName = ‘CustomerPortal’ and pageUrl like ‘%Checkout%’ AS ‘Begin Checkout’, <br>
      WHERE appName = ‘CustomerPortal’ and pageUrl like ‘%OrderConfirmed%’ AS ‘Order Confirmed’)<br>

The following tracks conversions on the basis that a user is authenticated by the time they begin checkout (__PageViews__)
>FROM PageView SELECT funnel(customerId, <br>
      WHERE appName = ‘CustomerPortal’ and pageUrl like ‘%Checkout%’ AS ‘Begin Checkout’, <br>
      WHERE appName = ‘CustomerPortal’ and pageUrl like ‘%OrderConfirmed%’ AS ‘Order Confirmed’)<br>

The same query as above but for a single page application (__AjaxRequests__)
>FROM AjaxRequest SELECT funnel(customerId, <br>
      WHERE appName = ‘CustomerPortal’ and requestUrl like ‘%Checkout%’ AS ‘Begin Checkout’ and httpResponseCode >= 200 and httpResponseCode < 300, <br>
      WHERE appName = ‘CustomerPortal’ and requestUrl like ‘%OrderConfirmed%’ AS ‘Order Confirmed’ and httpResponseCode >= 200 and httpResponseCode < 300)<br>

The funnel query pulls from a single data type. You won’t be able to combine PageViews and AjaxRequests in the same funnel.  If your bottom of the funnel flow includes a mix of PageViews and AjaxRequests you can capture the overall conversion rate by focusing on the PageViews at the start and the end.

## Synthetics Check
Change BOTF_MONITOR_NAME to match the synthetics monitor that validates the bottom of the funnel.  This is covered in the bottom of the funnel [implementation guide](https://docs.newrelic.com/docs/new-relic-solutions/observability-maturity/customer-experience/bofta-implementation-guide/#create-a-scripted-synthetics-check-for-the-bottom-of-the-funnel).

## Revenue at Risk - Latency
REVENUE_AT_RISK_LATENCY_FILTER - You can filter both related pages and Ajax Requests using pageUrl.  The [pageUrl value in an AjaxRequest](https://docs.newrelic.com/attribute-dictionary/?dataSource=Browser+agent&event=AjaxRequest&attributeSearch=pageUrl) reflects the page the user was on when they initiated the request.  

CONVERSION_VALUE - set this to what your organization considers the average value of a conversion.  Conversely, using custom attributes, you can change the query to be more specific and use the value of what the end user is about to purchase.   

For example, instead of <br>
>SELECT count(*) * 6.0 AS USD FROM PageView, AjaxRequest WHERE (pageUrl LIKE '%checkout%' or pageUrl LIKE '%confirmOrder%') AND (timeToSettle ..

where 6.0 is the average value of a conversion

You could use
>SELECT sum(cartValue) AS USD FROM PageView, AjaxRequest WHERE (pageUrl LIKE '%checkout%' or pageUrl LIKE '%confirmOrder%') AND (timeToSettle ..

provided that you have captured the cart value using [setCustomAttribute](https://docs.newrelic.com/docs/browser/new-relic-browser/browser-agent-spa-api/setcustomattribute-browser-agent-api/) or [setAttribute](https://docs.newrelic.com/docs/browser/new-relic-browser/browser-agent-spa-api/setattribute-browser-spa-api/).

One thing to be aware of is that in this example, the same cart value could be added multiple times if the same user experiences multiple interactions with slowness or errors.

## Revenue at Risk - Errors
REVENUE_AT_RISK_ERRORS_FILTER - This is likely to be the same as REVENUE_AT_RISK_LATENCY_FILTER

CONVERSION_VALUE - Revenue at Risk - Latency





 