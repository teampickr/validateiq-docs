![alt text](assets/logo.png "ValidateIQ Logo")

# Covid 19 Essential Work Sign Analysis - Code 3

This is the README for ValidateIQ analysis code 3 (Covid 19 Essential Work Sign Analysis). You can use this analysis to ensure that a given collection of images contains a minimum number of give essential work signs.

## Example Request

In order to detect the signs in your images you need to send a POST request to https://api.validateiq.com/iq/analysis with a webhook url which a analysis response can be sent to. This will allow you to process the results of you request.

An example JSON payload with code 3 (Covid 19 Essential Work Sign Analysis) to detect 1 sign would be:

```
{
    "AnalysisId": "6f056479-187a-4f71-a87d-dff53a07cd68",
    "Code": 3,
    "WebhookUrl": "https://my-webhook.url/",
    "WebhookSecret": "my-super-secret-code",
    "ImageUrls": ["... image url ..."],
    "Quantity": 1 // This is how many signs you want to find.
}
```

## Example Webhook Response

The webhook result will be sent to you as POST HTTP method.

An example of a response payload where the required amount of essential work signs are present:

```
{
   "Secret":"my-super-secret-code",
   "AnalysisId":"6f056479-187a-4f71-a87d-dff53a07cd68",
   "Code":3,
   "Version": "v1",
   "Quantity: 1
   "Result":{
      "CheckResponses":[
         {
            "Name":"Essential_Work_Sign",
            "Weight":1,
            "Score":86.41125
         }
      ],
      "OverallScore":86.41125,
      "ManualCheckRequired":false,
      "ManualCheckReason": null,
      "QuantityResults":[
         {
            "Name": "Essential_Work_Sign",
            "Quantity: 1
         }
      ]
   }
}
```

An example of a response payload where the required amount of essential work signs are not present:

```
{
   "Secret":"my-super-secret-code",
   "AnalysisId":"6f056479-187a-4f71-a87d-dff53a07cd68",
   "Code":3,
   "Version": "v1",
   "Quantity: 1
   "Result":{
      "CheckResponses":[
         {
            "Name":"Essential_Work_Sign",
            "Weight":1,
            "Score":26.22
         },
         {
            "Name":"Essential_Work_Sign",
            "Weight":1,
            "Score":46.32
         }
      ],
      "OverallScore":36.27,
      "ManualCheckRequired":true,
      "ManualCheckReason": "Minimum quantity not reached.",
      "QuantityResults":[
         {
            "Name": "Essential_Work_Sign",
            "Quantity: 0
         }
      ]
   }
}
```
