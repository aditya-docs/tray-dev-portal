Tray's APIs allow you to leverage the power of Tray's service connectors, without having to make use of the Builder UI. This gives you tremendous power to integrate powerful service operations into your own apps. 

The following APIs are available: 

- **Auths API** for creating, retrieving and deleting **authentications** for particular services
- **Connectors API** for calling **individual operations** for particular service connectors
- **Embedded API (GraphQL)** for managing and displaying the **Solutions and Instances which drive your Tray integrations** 

## Use cases

Some use cases include:

- **Storing authentications** which can be re-used in **multiple Tray integrations**, negating the need to request auths from End Users more than once

- **Calling authentications** to be used in external (i.e. non-Tray) integrations

- **Simple transfer of existing authentications** into Tray, to facilate transfer of your integrations

- **Using connectors and their operations** to power your integrations

- **Building your own workflow engine** a la the Tray builder