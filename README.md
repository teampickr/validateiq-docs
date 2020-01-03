![alt text](assets/logo.png "ValidateIQ Logo")

# What?

ValidateIQ is Pickr's intelligent machine learning powered tool for assessing evidence submitted for infrastructure projects. ValidateIQ will provide a score on how likely the submitted evidence is likely to be approved/signed off.

# How can I use it?

You must contact Pickr to get access to this API. Email us at hello[at]pickr[dot]works

# Usage

## Authentication

ValidateIQ's authentication is provided by Pickr's Identity service. You will need to contact this service to get an access token before you call call ValidateIQ API endpoints.

Access tokens will expire one hour after issuing. They are in JWT format. The endpoint to retrieve a token is `https://id.pickr.works/connect/token`

A cURL example:

```
curl -d "grant_type=client_credentials&client_id=My_Client_Id&client_secret=My_Client_Secret" -X POST https://id.pickr.works/connect/token
```

This will return the following JSON response:

```
{
    "access_token": "eyJhb...mJnd47jM",
    "expires_in": 3600,
    "token_type": "Bearer"
}
```

You will need to use the `access_token` for all API calls, and you will need to request a new token every hour.

**Add a `Authorization` header to all API requests with your access token prefixed with `Bearer`, for example:**

```
Authorization: Bearer eyJhb...mJnd47jM
```

## Submit an analysis request

ValidateIQ is an asynchronous API. When we receive your request it is queued and processed as quickly as possible. You have multiple options of how to be notified of the responses once your request has been processed. We will talk about those later.

To submit a new analysis request you need to provide the following properties in JSON format:

|Property|Description|Required
|:--|:--|:-:|
|AnalysisId|This is an ID provided by your system, it must be unique on a per code basis. For example, you can use the same analysis ID across multiple codes. This value can only be alphanumeric plus hyphens.|✅
|Code|This is the code of the checks you wish to be performed. Please contact us for a list of codes.|✅
|Base64Images|This is an array of base64 encoded images, in either PNG or JPEG.|✅
|WebhookUrl|This is a publicly accessible URL that you want us to `POST` your analysis results to once processed. You can use the `WebhookSecret` property to add a layer of protection.|
|WebhookSecret|This is a secret string that we will send back to you with the analysis results, you can use this to check the message is really coming from us.|
|Email|You can specify an email address that we should contact if a check fails to meet a 90% threshold.

An example JSON payload would be:

```
{
    "AnalysisId": "6f056479-187a-4f71-a87d-dff53a07cd68",
    "Code": 1,
    "WebhookUrl": "https://my-webhook.url/",
    "WebhookSecret": "my-super-secret-code",
    "Base64Images": ["... base64 encoded image..."]
}
```

An example cURL request would be:

```
curl -d "{ \"AnalysisId\": \"6f056479-187a-4f71-a87d-dff53a07cd68\", \"Code\": 1, \"WebhookUrl\": \"https:\/\/my-webhook.url\/\", \"WebhookSecret\": \"my-super-secret-code\", \"Base64Images\": [\"... base64 encoded image...\"] }" -H "Authorization: Bearer <token here>" -X POST https://api.validateiq.com/iq/analysis
```

A example successful response is (HTTP code 200):

```
{
    "id": "6f056479-187a-4f71-a87d-dff53a07cd68",
    "version": "v1",
    "code": 1
}
```

You can submit multiple requests with the same analysis ID and code, and we will create incrementing versions. Images from previous versions are copied across. This is useful for adding new images to an analysis. Each version will produce a response and webhook call/email.

# Webhook Responses

We recommend using webhooks as the best way to receive a response to your analysis requests.

Once we have processed your request we will immediate trigger a webhook response if you provided a url to us when making the request. If you do not provide a webhook secret we will default to an empty string.

Webhooks are send to you as POST HTTP method, with the following payload:

```
{
   "Secret":"my-super-secret-code",
   "AnalysisId":"6f056479-187a-4f71-a87d-dff53a07cd68",
   "Code":1,
   "Version": "v1"
   "Result":{
      "CheckResponses":[
         {
            "Name":"Dummy Check",
            "Weight":1,
            "Score":86.41125
         }
      ],
      "OverallScore":86.41125,
      "ManualCheckRequired":true,
      "ManualCheckReason":"Minimum threshold not reached."
   }
}
```

`CheckResponses` may contain additional properties depending on the check type, but will always contain `Name`, `Weight` and `Score`.

# Polling for a response

You can poll an endpoint to check if an analysis has been processed yet. We do not recommend this, we would suggest using a webhook.

The endpoint is:

`https://api.validateiq.com/iq/analysis/{id}-{code}/{version?}`

The version parameter is option, if omitted it will return the latest version.

An example cURL request would be:


```
curl -H "Authorization: Bearer <token here>" -X POST https://api.validateiq.com/iq/analysis/6f056479-187a-4f71-a87d-dff53a07cd68-1
```

or, to get a specific version:

```
curl -H "Authorization: Bearer <token here>" -X POST https://api.validateiq.com/iq/analysis/6f056479-187a-4f71-a87d-dff53a07cd68-1/v1
```

The result will be the same as the webhook above, with the addition of a `Secret` key which is the same as the `WebhookSecret` property you passed to us when making your request.


# Email notifications

You can provide an `Email` property when creating an analysis request. If the analysis result has an overall score of <90% then we will send you an email notification.
