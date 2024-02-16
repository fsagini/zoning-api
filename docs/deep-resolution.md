## Overview

The On demand Deep resolution API service allows users to run deep-resolution computations for a predefined area of interest, during a certain time period.

## Base URL

The URL for the On demand API is [https://api.digifarm.io/development/deep-resolution](https://api/digifarm.io/development/deep-resolution)

## Authentication

The On demand API uses client tokens for authentication. Include your client token in each request to the API.

This API provides deep-resolution data for a specific field over a specified date range. The endpoint accepts various query parameters for customization.

## **Endpoints**

### POST /deep-resolution/available_dates

This endpoint provides available dates for deep resolution imagery within a specified bounding box and time range.

#### Request URL `POST https://api.digifarm.io/deep-resolution/available_dates`

**Method**: POST

#### **Request Headers**

```
Content-Type: application/json
```

#### **Request Body**

*   `bbox`: \[object, required\] The bounding box coordinates.

    *   `x_min`: \[float, required\] The minimum longitude.
    *   `y_min`: \[float, required\] The minimum latitude.
    *   `x_max`: \[float, required\] The maximum longitude.
    *   `y_max`: \[float, required\] The maximum latitude.

*   `cloud_cover`: \[integer, required\] The maximum allowed cloud cover percentage.

*   `start_date`: \[string, optional\] The start date in ISO 8601 format

*   `end_date`: \[string, optional\] The end date in ISO 8601 format 

*   `client_token`: \[string, optional\] The client token for authorization.

*   `callback_url`: \[string, required\] The URL to call with the results.

    <br/>

**Example Request**

```
curl -X POST https://api.digifarm.io/deep-resolution/available_dates \
 -H "Content-Type: application/json" \
 -d '{
 "bbox": {
    "x_min": 11.16545302951809,
    "y_min": 60.73687126373886,
    "x_max": 11.21197326267199,
    "y_max": 60.75331374707744
 },
 "cloud_cover": 70,
 "start_date": "2023-10-01T00:00:00",
 "end_date": "2024-01-01T00:00:00",
 "client_token": "9ea17d0b-f837-4bab-9040-48277aa3d8f4",
 "callback_url": "https://webhook.site/e6249d0a-ed68-4b71-9d09-dc694acb34e4"
 }'
```

**Response**

*   `statusCode`: \[string\] The status of the request.

*   `message`: \[string\] A message describing the outcome of the request.

*   `version`: \[string\] API version number

*   `timestamp`: \[datetime\] The ID of the initiated task.

```json

{
    "statusCode":200,
    "data":{"message":"Fetching available dates, callback url will be called to https://webhook.site/e6249d0a-ed68-4b71-9d09-dc694acb34e4 when done"}
    "version":"v1.0",
    "timesetamp":"2023-08-02T03:17:23.340688"
}
```


### POST /deep-resolution

This endpoint provides deep resolution data for a specified bounding box over a given date range.

#### Request URL `POST https://api.digifarm.io/deep-resolution`

**Method**: POST

#### **Request Headers**

```
Content-Type: application/json
```

#### **Request Body**

*   `bbox`: \[object, required\] The bounding box coordinates.

    *   `x_min`: \[float, required\] The minimum longitude.
    *   `y_min`: \[float, required\] The minimum latitude.
    *   `x_max`: \[float, required\] The maximum longitude.
    *   `y_max`: \[float, required\] The maximum latitude.

*   `cloud_cover`: \[integer, required\] The maximum allowed cloud cover percentage.

*   `start_date`: \[string, optional\] The start date in ISO 8601 format

*   `end_date`: \[string, optional\] The end date in ISO 8601 format 

*   `client_token`: \[string, optional\] The client token for authorization.

*   `callback_url`: \[string, required\] The URL to call with the results.

*   `dates`: \[list of string, optional\] The dates to use for processing the area of interest.

* At least dates is required, or, alternatively, start_date and end_date.
** If start_date and end_date are provided, all available dates in between will be processed.


    <br/>

**Example Request**

```
curl -X POST "https://api.digifarm.io/development/deep-resolution" \
 -H "Content-Type: application/json" \
 -d '{
 "bbox": {
    "x_min": 11.16545302951809,
    "y_min": 60.73687126373886,
    "x_max": 11.21197326267199,
    "y_max": 60.75331374707744
 },
 "cloud_cover": 70,
 "start_date": "2023-10-01T00:00:00",
 "end_date": "2024-01-01T00:00:00",
 "client_token": "9ea17d0b-f837-4bab-9040-48277aa3d8f4",
 "callback_url": "https://webhook.site/e6249d0a-ed68-4b71-9d09-dc694acb34e4"
 }'
```

**Response**

*   `statusCode`: \[string\] The status of the request.

*   `message`: \[string\] A message describing the outcome of the request.

*   `version`: \[string\] API version number

*   `timestamp`: \[datetime\] The ID of the initiated task.

```json

{
    "statusCode":200,
    "data":{
    { "message":"Job queued",
      "job_id":"037d4b03-0b7a-46b5-b618-f81bdbe08739"
    },
    "version":"v1.0",
    "timesetamp":"2023-08-02T03:17:23.340688"
}
```

### POST /deep-resolution/status/{job_id}

This endpoint provides available dates for deep resolution imagery within a specified bounding box and time range.

#### Request URL `POST https://api.digifarm.io/deep-resolution/status/{job_id}`

**Method**: GET

#### **Request Headers**

```
Content-Type: application/json

client_token: \[string, required\] The client token for authorization.

```

    <br/>

**Example Request**

```
curl -X GET https://api.digifarm.io/deep-resolution/status/{job_id} \
 -H "Content-Type: application/json" \
```

**Response**

*   `statusCode`: \[string\] The status of the request.

*   `message`: \[string\] A message describing the outcome of the request.

*   `version`: \[string\] API version number

*   `timestamp`: \[datetime\] The ID of the initiated task.

```json

{
    "statusCode":200,
    "data":{
        "job_id":"e6249d0a-ed68-4b71-9d09-dc694acb34e4",
        "dates": []
    }
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

The On demand Webhook API provides a way for your application to interact with our service, notifying clients of the completion of tasks. It allows you to register URL endpoints (webhooks) to which we will send notifications regarding specific events happening within our system. When a job is completed, the API makes a POST request to the `callback_url` that was specified in the original request.

## Webhook Event

### Task Completion

When a task is completed, a POST request will be sent to the specified `callback_url`. Please ensure your API service is ready to accept POST requests at the callback URL you provide.

## Request Headers

*   `Content-Type`: `application/json`

*   `X-Digifarm-Signature`: A HMAC SHA256 signature computed from the payload and a shared secret, providing a way to ensure the request is from On demand API.

## Request Body

*   `job_id`: \[string\] The ID of the completed task.

*   `statusCode`: \[string\] The http status code of the task.

*   `data`: \[object\] The result of the task. This will be present if the task completed successfully.

```
{
    
    "statusCode":200,
    "data": {
        "job_id": "fdj4hi8e-3ds4-kl3d-fk4d-4dk3ji8e9sdf",
        "output":  "S3 download URL of the deep-resolution output in .zip format",
        "version":"v1.0",
        "timestamp": "[string] Time when the task was completed"
     },
    
}
```

## Security

Webhook delivery uses HTTPS to encrypt data during transmission. You may want to set up your endpoint to verify that incoming webhooks are from us.

### Verification

We will include a signature in each webhook event's HTTP headers. The header key is `X-Digifarm-Signature`. Your application can use this signature to verify the request was sent our On demand API. The value of this header is computed by generating a HMAC SHA256 hash of the payload using your client token as the key.

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

**Q: How can I verify that the request came from your service?** We include a signature in each HTTP header. You can use this signature to verify that the request came from our service.

<br/>
