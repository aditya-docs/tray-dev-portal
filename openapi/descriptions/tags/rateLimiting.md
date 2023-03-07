| Endpoint | Request rate limited? | Notes | Paginated response? | Notes |
|---|---|---|---|---|
| Get connectors | No |  | No |  |
| Get connector operations | No |  | No |  |
| Call connector | By 3rd party | A Tray 200 response will contain 3rd party errors with rate limiting details <br /><br /> Adding e.g. a 1 min delay should resolve any issues <br /><br /> Please also see 3rd party docs for details on rate limiting policies | By 3rd party | A Tray 200 response will contain 3rd party info re batch size, next page tokens etc.  |
| Create user authentication | No |  | No |  |
| Get user authentication (by id) | Yes | You should cache user authentications until just before they expire | No |  |
| Delete user authentication | No |  | No |  |
| Decrypt user authentication | Yes |  | No |  |
| Get service environments | No |  | No |  |

## Notes on 3rd party pagination

Certain 3rd party operations such as Salesforce 'List records' or Marketo 'List leads' may return long lists of data which will need to be paginated.

This can be dealt with using parameters such as ‘batch_size’ ‘next_page_token’ etc.

**The exact pagination parameters will depend on the service**.

Next page tokens and batch sizes will have to be **passed in the call connector input**.

The **token for the next batch** and e.g. ‘has more’ information will be **returned in the 200 response**.

You can use our <a href="https://connectivity-api-demo.adityasingh239.repl.co/" target="_blank">operations explorer</a> to check the input and output schema for each operation to look for any pagination parameters.

The following shows an example input for the Salesforce 'List Records' operation which passes a `batch_size` and `page_offset` token:

```
{
    "operation": "find_records",
    "authId": "a5f886xx-xxxx-xxx-xxx-xxf9459933",
    "input": {
        "object": "Account",
        "batch_size": 200,
        "page_offset": "0r84J1RcJ9gaMpWQKU-200",
        "fields": [
            "Id",
            "Name"
        ],
        "conditions_type": "Match any conditions"
    },
    "returnOutputSchema": false
}
```    

## Notes on DDL pagination

You may find that some DDL operations (identified as 'lookup' in the input schema) do not have any pagination parameters.

For example the Slack 'send_message' operation has a 'list_users_and_conversations_ddl' to allow the user to select the channel they wish to send the message to.

However if you have more than 1000 channels (the max returned by the Slack API) then you will be unable to paginate with the DDL operation.

In this case, it is recommended that you make use of the <a href="https://connectivity-api-demo.adityasingh239.repl.co/" target="_blank">operations explorer</a> to find a non-DDL version of the operation.

In this case you can make use of the 'list_conversations' operation, which accepts the `limit` and `cursor` pagination parameters:

```
{
    "operation": "list_conversations",
    "authId": "af75fxxx-xxx-xxx-xxx8c494c5",
    "input": {
        "limit": 10,
        "cursor": "dGVhbTpDRTZEN0s2UTM="
    },
    "returnOutputSchema": false
}
    
```    

After using this to allow a user to select the channel you can pass their choice as a direct input for `channel` in the 'send_message' operation:

```
{
    "operation": "send_message",
    "authId": "af75fxxx-xxx-xxx-xxx8c494c5",
    "input": {
        "channel": "{{channel}}",
        "text": "more please!",
        "as_user": true
    },
    "returnOutputSchema": false
}     
```
