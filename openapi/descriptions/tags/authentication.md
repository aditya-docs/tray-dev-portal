___

## User Model

Before creating authentications to call connectors with, it is necessary to **create the users that will own these authentications**

**Each of these users can own multiple authentications** and will have their own Tray <a target="_blank"  href="https://tray.io/documentation/platform/accounts/workspaces/overview/">personal workspace</a>.

Tray does not have strict limits on the creation of users.

A **one-to-one mapping** between your system’s users and Tray users is the recommended approach.
**Tray features are designed assuming this is the case.**
The following table shows how all aspects of user management are dealt with:

| Feature | How it works | Notes | API used |
|---|---|---|---|
| User creation | via Embedded API | should be the users in your company that are part of the system being integrated with Tray | Create User API |
| Auth ownership | a user can own multiple authentications | otherwise solutions with more than one authentication will not be able to be executed | Create User Auth - but normally created via auth-only dialog (see below) |
| Permission model | based on workspaces | can have an impact on embedded solutions that use workspace objects such as authentications or connectors | n/a |
| Auth management | systems can read or delete all a user's auths with a single API call | could be triggered after a user is removed from the system | Get User Auths and Delete User Auth |


The simplest way to integrate Tray users would be to add a field to the entities that represent the concept of a user in the system’s model.

It could also be modelled as an “add on” feature without changing the system’s domain by:

- adding the mapping as a new concept that would link a local user with a tray user
- these user mappings can then be created as part of the creation of the authentication:
  - if the user mapping doesn’t exist create it
  - otherwise reuse the one that already exists
            
<br />
<img src="https://images.ctfassets.net/p3dvvupz8xw4/31Q2PZPObMn1KIyrMSWG9i/b5277a9596312710b88517131b1ae7f0/connectivity-api-user-model.png?w=1080&q=100" />

Of course these are just suggestions that may or may not apply depending on your internal user model

## Creating authentications

**Prerequisites**

You will need a <a target="_blank" href="https://tray.io/documentation/embedded/getting-started/embedded-id-and-master-token/">master token</a> to create End Users, obtain serviceIds and create auths.

<br />
<strong>Getting connector serviceIds</strong>
<br />
<br />
<button class="accordion-button">1 - Get service version</button>
<div class="accordion-body">

**https://api.tray.io/core/v1/connectors**
    
The Get Connectors endpoint will return all of the connectors that you have access to.
    
The response includes: 
- connector name
- connector version
    
These can then be used with the Get Connector Operations endpoint to list the available operations within a particular
connector. So if your organisation has access to Outreach and Twilio you would see the following:
    
    {
        "elements": [
        {
            "name": "outreach",
            "version": "3.3",
            "title": "Outreach",
            "description": "",
            "service": {
                "name": "outreach",
                "version": 2
            }
        },
        {
            "name": "twilio-output",
            "version": "2.1",
            "title": "Twilio",
            "description": "Cloud communication platform for developers",
            "service": {
                "name": "twilio",
                "version": 1
            }
        }
        ]
    }
</div>
<button class="accordion-button">2 - Get serviceId, serviceEnvironmentId</button>
<div class="accordion-body">

**https://api.tray.io/core/v1/services/{serviceName}/versions/{serviceVersion}/environments**

Before creating an authentication for a service you will need to retrieve its **serviceId** and **serviceEnvironmentId** with the Get service
environments endpoint:

    {
        "elements": [
        {
            "id": "3e85ecdb-0e17-48d6-b59b-0132f01af393",
            "title": "",
            "authenticationType": "token",
            "formDataSchema": {},
            "dataSchema": {
            "$schema": "http://json-schema.org/draft-04/schema#",
            "id": "http://jsonschema.net",
            "type": "object",
            "properties": {
                "api_key": {
                    "id": "http://jsonschema.net/api_key",
                    "title": "API key",
                    "type": "string"
                }
            }
        },
        "additionalProperties": false,
        "required": ["api_key"]
        }
        ]
    }
</div>
<br />
<strong>Creating users and user tokens</strong>
<br />
<br />
<button class="accordion-button">1 - Create an end user</button>
<div class="accordion-body">

Your end users will own the authentications and instances that we will create later.
This endpoint accepts an external identifier so that you can link the created Tray user with an identifier in your
application.

    mutation {
        createExternalUser(input: {name: "Billy Bluehat", externalUserId: "96xxxxxxd7"}) {
            userId
        }
    }

The endpoint returns a userId that will be used to identify the user when using Tray APIs.

    {
        "data": {
            "createExternalUser": {
                "userId": "fbb96559-xxxx-xxxx-xxxx-5552c2d2fca4"
            }
        }
    }
</div>
<button class="accordion-button">2 - Create a User access token</button>
<div class="accordion-body">

Once a user has been created, <a target="_blank"
    href="https://embedded-api-docs.tray.io/?version=latest#79a5484c-65d3-4181-89df-875c7f2c3ebb">this endpoint</a> can
be used to generate an access token for the created user.

The master token obtained in the first step and the userId for the user created in the previous step are used in the
request below to obtain the access token.

    mutation {
        authorize(input: {userId: "fbb96559-xxxx-xxxx-xxxx-5552c2d2fca4"}) {
            accessToken
        }
    }

The access token obtained here is used to call the connector call API. **This token will be valid for 2 days**.

    {
        "data": {
            "authorize": {
                "accessToken": "0c8c8e16e39e4xxxxxxxxxxxxxxxxxxxxxx49545b584e307063710d1ee"
            }
        }
    }
</div>
<br />
<strong>Auth option 1: creating new End User auths</strong>
<p>If you need to prompt your End Users to create new auths from scratch, you can make use of Tray's **auth-only dialog**:</p>
<button class="accordion-button">1 - Generate an authorization code for the user</button>
<div class="accordion-body">

In order to open the auth collection dialogue so that the user can provide their authentication details, an
authorization code needs to be obtained using the <a href="https://embedded-api-docs.tray.io/?version=latest#35b36670-c570-4fe2-adb5-55a34658ccab">Create Config Wizard Authorization Code endpoint</a>:

    mutation {
        generateAuthorizationCode(input: {userId: "fbb96559-xxxx-xxxx-xxxx-5552c2d2fca4"}) {
            authorizationCode
            clientMutationId
        }
    }

This endpoint requires the **master token** and the **userId** created in <a href="#step2">step2</a>. The code
returned by this endpoint is single-use and expires after 5 minutes.

    {
        "data": {
            "generateAuthorizationCode": {
                "authorizationCode": "b8aab26cxxxxxxxxxxxxxxxxxxxx776c7bba7977",
                "clientMutationId": null
            }
        }
    }
</div>
<button class="accordion-button">2a - Assemble the auth collection url</button>
<div class="accordion-body">

You can now assemble the URL that will be passed to your user to collect authentication details for a particular
connector:

`https://embedded.tray.io/external/auth/create/{partnerName}?code={authorizationCode}&serviceId={serviceId}&serviceEnvironmentId={serviceEnvironmentId}&scopes[]={scopes}`

Ideally, you should direct your user to a popup window with this URL.

- **partnerName** is the 'Embedded ID' that can be set/viewed in the embedded settings page (seen here)
- **authorizationCode** is the single use authorization code generated in the previous step
- **serviceId** is a UUID which identifies the Tray authentication service for the connector that you are collecting
auth details for
- **serviceEnvironmentId** is a UUID that identifies the services environment that the authentication service belongs to
- **scopes** are the OAuth scopes required to ensure that the necessary permissions are set once the OAuth flow is
completed:
- Each scope needs to be passed as an individual parameter so should look like this:
`&scopes[]=channels:read&scopes[]=channels:write`
</div>
<button class="accordion-button">2b - Example collection url</button>
<div class="accordion-body">
A valid auth collection URL would look like this:

`https://embedded.tray.io/external/auth/create/traydemoaccount?code=5fb286bb55ed8763f409dc434ceed3484c63201f&serviceId=55d1154c-4fd8-4c2c-ba22-041deabfcbfd&serviceEnvironmentId=973b1afa-aa13-42c7-86c3-3964f44c53dd`

The service ID and service environment ID correspond to the Twilio connector.

No scopes are required because the Twilio connector uses token based authentication.

The URL will take your user to a dialogue where they can provide their Twilio credentials:
<img src="https://images.ctfassets.net/p3dvvupz8xw4/7oNSulZJQ3BuQ5Ougbe05C/bcb4dcf7cd4ac45a9f95aae528c71d7e/create-twilio-auth.png?w=3840&q=100" alt="auth collection url" />
</div>
<br />
<strong>Auth option 2: import existing auths</strong>
<p>If you have pre-existing auths stored externally, you can import them to Tray with the Create authentication endpoint.
These auths can then be used for Tray 'internal' integrations or for your self-administered 'external' integrations:
</p>
<button class="accordion-button">Create auth endpoint</button>
<div class="accordion-body">
    
**https://api.tray.io/core/v1/authentications**
    
The following shows an example of creating an auth:
    
    {

        "scopes": [],
        "workspaceId": "27cdffxxx-xxx-xxx-xxx29e5f2a",
        "formData": {},
        "data": {
            "api_key": "keyxxxxxxxdC5dK"
        },
        "serviceEnvironmentId": "3e85ecdb-0e17-48d6-b59b-0132f01af393",
        "name": "MikeH Airtable staging test auth 2"
    }

This will return an `authenticationId` and an `externalId`:

    {
        "authenticationId": "21c6axxx-xxx-xxx-xxxde3d6c",
        "externalId": "a229axxx-xxx-xxx-xxxc49da552"
    }
</div>

## Using authentications

Stored auths can then be used in the following ways:

- **For your external self-hosted integrations** you can retrieve auths using the https://api.tray.io/core/v1/authentications/{{authId}} endpoint

- **For your Tray Embedded integrations** you can:

    - **Create Solution Instances** for your End Users, whereby you attach the authId to the services involved in their integration

    - **As a generic auth** which will be used by all End Users of your integrations

## Auth refresh

**External integrations**

For all stored authentications, Tray will manage the refresh of tokens on your behalf.

The following points should be noted when you are **storing auths for use with external (i.e. non-Tray) integrations**:

- The `expiresIn` property can be retrieved from the '' endpoint
- When using auths externally, you should cache your auth token and get a new token from Tray shortly before the expiry date
- Tray will always return a refreshed token
- The Get authentication endpoint is subject to rate limiting

**Tray Embedded integrations**

For internal Tray Embedded integrations, Tray will manage refresh **on your behalf**.

