When building Tray integrations, you will need to consider the following components for whitelabelling purposes:

- **Webhook urls** that your End Users might need to trigger instance runs can be whitelabelled using **\*.integration-hook.com** in place of **\*.trayapp.io** 

- For the **Config Wizard** you will need to create a **custom auth app** in the admin UI for each of the third party services you are using:

    - The **launch url** can be customized with **\*.integration-configuration.com** to replace **embedded.tray.io**
    - the authentication **redirect url** which you will need to enter for your custom auth app can be customized with **\*.integration-authentication.com**

Creating a Custom Auth app will allow you to replace all visual mentions of Tray in the Config Wizard with your own company and logo.

Note that creating an auth app in the builder is not done in the properties panel, but in the new auth dialog.

Please see our <a href="https://tray.io/documentation/embedded/core-topics/whitelabelling/intro/" target="_blank"> section on whitelabelling for more info </a>

Please also see individual service connector pages for specific notes and guidance on creating auth apps for those services.

A couple of points to note are:

- It is recommended that you **do not set the permission scopes for a custom auth app when creating it in the 3rd party UI**. Instead you should use the standard method of **setting the scopes when you are creating an authentication in your source / master workflow for each Solution**. This way you can have **multiple integrations that use the same custom auth app** and control them within Tray

- It is **not currently possible to share service environment IDs across Tray Orgs** so you will need to create separate auth apps for e.g. a dev org and a prod org

Any custom auth apps you create will then have a **serviceId** which can be retrieved using the 'Get service environments' endpoint:

https://api.tray.io/core/v1/services/{serviceName}/versions/{serviceVersion}/environments
