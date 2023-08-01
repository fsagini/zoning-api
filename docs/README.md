
## **POST /zoning/**

## Overview

The Zoning API service allows users to run zoning computations for a predefined area of interest, during a certain time period.

## Base URL

The URL for the Zoning API is [https://api/digifarm.io/development/zoning](https://api/digifarm.io/development/zoning)

## Authentication

The Zoning API uses client tokens for authentication. Include your client token in each request to the API.

This API provides zoning data for a specific field over a specified date range. The endpoint accepts various query parameters for customization.

## **Endpoints**

### POST /zoning

This endpoint provides timeseries data for a specified field over a given date range.

#### Request URL `POST https://api.digifarm.io/development/zoning`

**Method**: POST

#### **Request Headers**

```
Content-Type: application/json
```

#### **Request Body**

*   `num_zones`: \[integer, required\] The number of zones.

*   `geojson`: \[string, required\] The GeoJSON data as a string.

*   `years`: \[list of integers, required\] The years to include in the analysis.

*   `months`: \[list of integers, required\] The months to include in the analysis.

*   `callback_url`: \[string, required\] The URL to call with the results.

*   `client_token`: \[string, required\] The client token for authorization.
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

*   `status`: \[string\] The status of the request.

*   `message`: \[string\] A message describing the outcome of the request.

*   `task_id`: \[string\] The ID of the initiated task.

```json
{
    "status": "success",
    "message": "Task initiated successfully. You will be notified at the callback URL when the task is complete.",
    "task_id": "fdj4hi8e-3ds4-kl3d-fk4d-4dk3ji8e9sdf"
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
    "status": "error",
    "message": "Invalid input data. Please provide all fields in correct format."
}
```

**Condition**: If the 'client\_token' is missing or invalid.

**Code**: 401 UNAUTHORIZED

**Content Example**:

json

```
{
    "status": "error",
    "message": "Invalid client token."
}
```

**Condition**: If there's a problem initiating the task.

**Code**: 500 INTERNAL SERVER ERROR

**Content Example**:

json

```
{
    "status": "error",
    "message": "Could not initiate task due to an internal server error. Please try again later."
}
```

<br/>

# Webhook Service

## Overview

The Zoning API uses webhooks to notify clients of the completion of tasks. Webhooks deliver data to other applications as it happens, meaning you get data immediately. When a task is completed, a POST request will be made to the `callback_url` that was specified in the original request.

## Webhook Event

### Task Completion

When a task is completed, a POST request will be sent to the specified `callback_url`.

## Request Headers

*   `Content-Type`: `application/json`

*   `X-Digifarm-Signature`: A HMAC SHA256 signature computed from the payload and a shared secret, providing a way to ensure the request is from Zoning API.

## Request Body

*   `task_id`: \[string\] The ID of the completed task.

*   `status`: \[string\] The status of the task. This will be either `success` or `error`.

*   `message`: \[string\] A message describing the outcome of the task.

*   `data`: \[object\] The result of the task. This will be present if the task completed successfully.

```
{
    "task_id": "fdj4hi8e-3ds4-kl3d-fk4d-4dk3ji8e9sdf",
    "status": "success",
    "message": "Task completed successfully.",
    "data": {
    	"raster": "[string] URL of the raster result",
    	"geojson": "[string] URL of the geojson result",
    	"timestamp": "[string] Time when the task was completed"
     }
}
```

## Responding to a webhook

Your endpoint should return a `200 OK` response to acknowledge that it received the webhook event. If the Zoning API does not receive a `200 OK` response, it will attempt to send the webhook event again.

<br/>

## Verifying webhook requests

Webhook POST requests include a `X-Digifarm-Signature` header which you can use to verify that the request was sent by the Zoning API. The value of this header is computed by generating a HMAC SHA256 hash of the payload using your client token as the key.

Here's an example in Python showing how you could verify a webhook request:

```python
import hmac
import hashlib

def is_valid_signature(x_digifarm_signature, data, client_token):
    expected_signature = hmac.new(
        client_token.encode(),
        msg=data.encode(),
        digestmod=hashlib.sha256
    ).hexdigest()

    return hmac.compare_digest(expected_signature, x_digifarm_signature)
```

<br/>

### Error Handling

If there's an error during the processing of the task, the webhook request will still be sent, but the `status` field will be `error` and the `message` field will contain information about the error. The `data` field will not be present in this case.

**Condition**: If there's a problem processing the callback request.

**Code**: 400 BAD REQUEST

**Content Example**:

```
{
    "task_id": "fdj4hi8e-3ds4-kl3d-fk4d-4dk3ji8e9sdf",
    "status": "error",
    "message": "There was an error processing the task."
}
```

Please ensure your API service is ready to accept POST requests at the callback URL you provide.
