## **POST /zoning?clientToken={token}**

## Overview

The Zoning API service allows users to run zoning computations for a predefined area of interest, during a certain time period.

## Base URL

The URL for the Zoning API is [https://api.digifarm.io/development/zoning](https://api/digifarm.io/development/zoning)

## Authentication

The Zoning API uses client tokens for authentication. Include your client token in each request to the API.

This API provides zoning data for a specific field over a specified date range. The endpoint accepts various query parameters for customization.

## **Endpoints**

### POST /zoning

This endpoint provides zoning data for a specified field over a given date range.

#### Request URL `POST https://api.digifarm.io/development/zoning?token=""`

**Method**: POST

#### **Request Headers**

```
Content-Type: application/json
```

#### **Request Parameters**

*   `client_token`: \[string, required\] The client token for authorization.

#### **Request Body**

*   `num_zones`: \[integer, required\] The number of zones.

*   `geojson`: \[string, required\] The GeoJSON data as a string.

*   `years`: \[list of integers, optional\] The years to include in the analysis.

*   `start_year`: \[string, optional\] The years to include in the analysis.

*   `end_year`: \[string, optional\] The years to include in the analysis.

*   `months`: \[list of integers, optional\] The months to include in the analysis.

*   `callback_url`: \[string, required\] The URL to call with the results.

    <br/>

**Example Request**

```
curl -X POST "https://api.digifarm.io/development/zoning" \
-H "Content-Type: application/json" \
-d '{
 "num_zones": 5,
 "geojson": "{\"type\":\"FeatureCollection\",\"name\":\"sm1\",\"crs\":{\"type\":\"name\",\"properties\":{\"name\":\"urn:ogc:def:crs:OGC:1.3:CRS84\"}},\"features\":[{\"type\":\"Feature\",\"properties\":{},\"geometry\":{\"type\":\"Polygon\",\"coordinates\":[[[-58.925683731259305,-32.170670143149806],[-58.921718286989808,-32.165855068462285],[-58.918874139461558,-32.173590838544861],[-58.919670597018118,-32.173688600787514],[-58.919894375123128,-32.173816913571734],[-58.920127778092876,-32.174126492878308],[-58.920012279716104,-32.174238511447072],[-58.919612847829711,-32.174173337023817],[-58.919064230540023,-32.174175373725255],[-58.918633517843269,-32.174069465190371],[-58.918152274606669,-32.175377019629543],[-58.919490130804377,-32.175747693640915],[-58.919923249717321,-32.174541979241148],[-58.921289980509229,-32.174871922943709],[-58.922262091847138,-32.172370838399573],[-58.923927193445728,-32.172794479701082],[-58.924326625332114,-32.171922754109467],[-58.925260237211091,-32.172146796530058],[-58.925683731259305,-32.170670143149806]]]}}]}",
 "years": [2022],
 "months": [1,2,3],
 "callback_url": "https://73c0-41-90-65-251.ngrok-free.app/ping",
 "client_token": "165af233-6577-4a41-a6c9-c38013d9e18d"
}'
```

**Response**

*   `statusCode`: \[string\] The status of the request.

*   `message`: \[string\] A message describing the outcome of the request.

*   `job_id`: \[string\] The ID of the initiated task.

*   `version`: \[string\] API version number

*   `timestamp`: \[datetime\] The ID of the initiated task.

```json

{
    "statusCode":200,
    "data":{
    "message":"zoning request queued. You will be notified at the callback URL when the task is complete.",
    "job_id":"7f81740d-e8b5-473f-b2b5-3d75480adf70"
    },
    "version":"v1.0",
    "timesetamp":"2023-08-02T03:17:23.340688"
}
```

### **Error Response**

In case of errors, the API will return an HTTP status code and a JSON object containing an error message.

**Condition**: If fields are missing or the data is in the wrong format.

**Code**: 400 BAD REQUEST

**Content Example**:

Type json

```
{
  "statusCode":400,
  "data":{
      "message": "Bad request. Invalid input data. Please provide all fields in correct format."
  },
  "version":"v1.0",
  "timestamp": "Current timestamp"
}
```

**Condition**: If the 'client\_token' is missing or invalid.

**Code**: 401 UNAUTHORIZED

**Content Example**:

json

```
{
  "statusCode":401,
  "data":{
      "message": "Unauthorized. Invalid client token."
  },
  "version":"v1.0",
  "timestamp": "Current timestamp"
}
```

**Condition**: If there's a problem initiating the task.

**Code**: 500 INTERNAL SERVER ERROR

**Content Example**:

json

```
"statusCode":500,
"data":{
    "message": "Internal Server Error. Could not initiate task due to an internal server error. Please try again later."
},
"version":"v1.0",
"timestamp": "Current timestamp"
```

# Webhook Service

## Overview

The Zoning Webhook API provides a way for your application to interact with our service, notifying clients of the completion of tasks. It allows you to register URL endpoints (webhooks) to which we will send notifications regarding specific events happening within our system. When a job is completed, the API makes a POST request to the `callback_url` that was specified in the original request.

## Webhook Event

### Task Completion

When a task is completed, a POST request will be sent to the specified `callback_url`. Please ensure your API service is ready to accept POST requests at the callback URL you provide.

## Request Headers

*   `Content-Type`: `application/json`

*   `X-Digifarm-Signature`: A HMAC SHA256 signature computed from the payload and a shared secret, providing a way to ensure the request is from Zoning API.

## Request Body

*   `job_id`: \[string\] The ID of the completed task.

*   `statusCode`: \[string\] The http status code of the task.

*   `data`: \[object\] The result of the task. This will be present if the task completed successfully.

```
{
    
    "statusCode":200,
    "data": {
        "job_id": "fdj4hi8e-3ds4-kl3d-fk4d-4dk3ji8e9sdf",
        "output":  "S3 download URL of the zoning output in .zip format",
        "version":"v1.0",
        "timestamp": "[string] Time when the task was completed"
     },
    
}
```

## Security

Webhook delivery uses HTTPS to encrypt data during transmission. You may want to set up your endpoint to verify that incoming webhooks are from us.

### Verification

We will include a signature in each webhook event's HTTP headers. The header key is `X-Digifarm-Signature`. Your application can use this signature to verify the request was sent our Zoning API. The value of this header is computed by generating a HMAC SHA256 hash of the payload using your client token as the key.

Here's an example in Python showing how you could verify a webhook request:

```python
import hmac
import hashlib

def is_valid_signature(x_digifarm_signature, data, client_token):
    payload_str = json.dumps(data.dict(), sort_keys=True)
    expected_signature = hmac.new(
        client_token.encode(),
        msg=payload_str.encode(),
        digestmod=hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected_signature, x_digifarm_signature)

```

## Retries and Exponential Backoff

Your endpoint should return a `200 OK` response to acknowledge that it received the webhook event. If the Zoning API does not receive a `200 OK` response, it will attempt to send the webhook event again. You should process the webhook payload asynchronously. This is because webhook endpoint has a timeout of 10 secs, so any extensive processing that blocks the response may result in timeouts and unnecessary webhook retries.

In case of delivery failure, our service will retry sending the webhook. Retries use an exponential backoff strategy, which means the delay between retries will progressively get longer to a maximum of 24 hours.

When our service encounters an error while delivering an event, we will attempt to retry the delivery. If the delivery fails after several attempts, the event delivery is considered failed. We highly recommend monitoring these failed deliveries to ensure no critical updates are missed.

### Error Handling

If there's an error during the processing of the task, the webhook request will still be sent, but the `statusCode` field will be `error` and the `message` field will contain information about the error.

**Condition**: If there's a problem processing the callback request.

**Code**: 400 BAD REQUEST

**Content Example**:

```
"statusCode":400,
"data":{
    "data": {
        "job_id": "fdj4hi8e-3ds4-kl3d-fk4d-4dk3ji8e9sdf",
        "version": "v1.0",
        "error": "Bad Request. There was an error processing the task.",
        "timestamp": "Current timestamp"
    },
},
```

## FAQ

**Q: What HTTP status codes should I return?** Our service considers any status code between 200 and 299 as a success. Other status codes will be considered as a failure, and we will retry the delivery.

**Q: What if my endpoint is temporarily down?** Our service will retry failed deliveries using an exponential backoff strategy.

**Q: How can I verify that the request came from your service?** We include a signature in each HTTP header. You can use this signature to verify that the request came from our service.

<br/>
