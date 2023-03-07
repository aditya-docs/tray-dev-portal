Executes an operation of a connector and returns the result as the response.

Please see the following documentation for detailed guidance on calling connectors:

- <a href="https://tray.io/documentation/platform/connectivity-apis/authentication/" target="_blank">Authentication</a>
talks you through the user model and how to obtain an **authId** required to call connector operations on behalf of your
end users
- <a href="https://tray.io/documentation/platform/connectivity-apis/using-connectors-and-operations/"
    target="_blank">Using connectors and operations</a> talks you through how to discover and call specific operations
with an authId and format correct input bodies
- <a href="https://tray.io/documentation/platform/connectivity-apis/example-use-case/" target="_blank">Example use
    case</a> shows a simple mockup UI of how you might use the Connectivity API endpoints to present an application to
your end users
- <a href="https://tray.io/documentation/platform/connectivity-apis/dynamic-ddl-operations//" target="_blank">Dynamic
    (DDL) inputs</a> talks you through how to deal with operations which are dependent on results from other operations

The input is a regular json object that must fit the input schema of the operation.
The authId field should be the id of a previously created authentication.

The output is also a regular json object containing the response from the connector.
The outcome field indicates whether or not the execution was successful.

If the execution failed because of a problem in the 3rd party service the value of
outcome will be "error" and the output will contain error information from
the 3rd party service, note that in this case the response to this request will still
be 200 OK, 500 or 400 status codes will be returned only if there was a problem
with the call itself, not with the 3rd party services.