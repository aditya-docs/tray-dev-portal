___

## Responses and errors

It is important to understand that responses can come from **either Tray or the 3rd party API**.

- All **non-200** responses (400 Invalid input, 401 Unauthorized etc.) will be responses **from Tray**

- A 200 response indicates that Tray completed a call to the 3rd party:

Therefore a **200 response does not necessarily indicate success** and will contain any success or failure responses from the 3rd party.

For example the following call to Slack which attempts to send a message to a channel which does not exist:


    {
        "operation": "send_message",
        "authId": "af75f833-a36b-4d73-a4a3-497958c494c5",
        "input": {
            "channel": "test200",
            "text": "more please!",
            "as_user": true
        },
        "returnOutputSchema": false
    }

Will return a status of 200, as the input met Tray's schema requirements, but within the 200 response we will see that Slack has returned an error:

    {
        "outcome": "error",
        "output": {
            "response": {
                "statusCode": 200,
                "body": {
                    "ok": false,
                    "error": "channel_not_found"
                }
            },
            "message": "Invalid response."
        }
    }



## Input schema issues

If you are having trouble getting the input schema exactly right for a particular operation, an effective approach is to run a test of the operation in the Tray builder UI and then inspect the logs to find exactly what is required in terms of structure, data types, escaped characters, minified strings etc.:

<img src="https://images.ctfassets.net/p3dvvupz8xw4/7kByJAlW6ZXjD3tQycuLSL/dfdac87bfab6ae42b60774b51881f14d/gsheet-get-rows-logs-input-schema.png" alt="gsheet-get-rows-logs-input-schema">

The above example shows how you can determine the 'filter' input requirements for the Google Sheets 'get rows' operation, which may be difficult to discern from the 'Get Connector Operations' response:

    "filter": {
        "type": "object",
        "properties": {
            "column_heading": {
                "type": "string",
                "description": "The column heading.",
                "lookup": {
                    "operation": "list_worksheet_column_headers_ddl",
                    "input": {
                        "spreadsheet_id": "{{spreadsheet_id}}",
                        "worksheet_name": "{{worksheet_name}}"
                    }
                },
                "title": "Column heading"
            },
            "operator": {
                "type": "string",
                "enum": [
                    "Equal to",
                    "Not equal to",
                    "Greater than",
                    "Less than",
                    "Greater than or equal to",
                    "Less than or equal to",
                    "Text contains",
                    "Text doesn't contain",
                    "Empty cells"
                ],
            ... 
            ... 
            ...


**Common Tray errors**

```
{
	"outcome": "error",
	"output": {
		"message": "Unfortunately, the connector unexpectedly failed. Please contact support if this persists. Error ID: d3e9899adc"
	}
}
```

Could be caused by an incorrect programmatic name for the operation e.g. `private_list_object_field` instead of `private_list_object_fields`