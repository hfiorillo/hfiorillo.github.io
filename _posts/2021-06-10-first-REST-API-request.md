---
title:  "Your first REST API Call in Azure"
date:   2020-06-10 00:00:00 +0000
categories: azure
tags:
    - azure
    - public cloud
    - REST APIs
    - devops
    - scripts
classes: wide
toc: true
header: 
    overlay_image: assets/images/your-first-api-request/microsoft-azure-blog-header.png
    overlay_filter: rgba(0, 0, 0, 0.4)
published: true

---

An introduction to Azure's REST API and a walkthrough demo.

---

In todays post, we're going to make a REST API call, both GET and PUT, towards an Azure API using an Azure API and a VS Code Extension called [Thunder Client](https://www.thunderclient.io/).

Thunder Client is a lightweight extension built for VS Code and is perfect for making a first API call, for those interested in more functionality and detail you can look at [Postman](https://www.postman.com/) or [Insomnia](https://insomnia.rest/).

### But first, why use REST API calls when working with Public Cloud? 💭

The more popular public cloud infrastructure becomes the important it becomes to manage at scale, whilst GUI's provide ample means to do so functionality can become limited and there are often problems that cannot be solved at scale through the portal without manual labour. We don't like manual labour in the cloud, instead we automate! We can do so by making use of REST API's. Learning to interact with resources via API requests, like IaC, can add another weapon to your arsenal when configuring, managing and creating cloud infrastructure. It is increasinly important to understand how to interact with REST API's as this can allow you to interact with them on a more complex level; perhaps incorporating them into scripts, applications or pipelines. What makes REST APIs so powerful is their flexibility and scalability, you are able to return a variety of resource information, make changes to resources and even create resources, through what is actually a very simple process (the API does all the hard work). REST APIs aren't limited to Azure, all major public cloud providers have their own functional API (AWS, Azure, Google Cloud & others!) and due to their standardised nature, the ability to interact with one can can help prepare you for the possibility your business goes multi-cloud!

# Your first REST API Call 👋

### Thunder Client

To get started with [Thunder Client](https://marketplace.visualstudio.com/items?itemName=rangav.vscode-thunder-client) simply install the VS Code extension from the Extensions Marketplace and you're set.

### Service Principal Creation

To do this we can use the az cli to make a service principal with contributor permissions.

1. First install the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
2. Then run this set of commands. Logging into your Azure account and creating the Service Principal role with Contributor permissions.

    ```c
    az login
    az ad sp create-for-rbac --role Contributor
    ```

3. Save the JSON output!

### Creating our Azure Environment in Thunder Client

1. In Thunder Client we will need to create a new Env which we will call Azure.
2. Click on the Thunder Client VS Code Icon on the left hand side, select **Env** and then the 3 lines here you can create a new Environment.

![/assets/images/your-first-api-request/Env.png](/assets/images/your-first-api-request/Env.png)

You will need to add the parameters listed below:

- `tenantId : <input your tenantId from the Service Principal you created>`
- `clientId : <input your clientId from the Service Principal you created>`
- `clientSecret: <input your clientSecret from the Service Principal you created>`
- `subscriptionId : <input your subscriptionId from the Service Principal you created>`
- `accessTokenURL :  [https://login.microsoftonline.com/{{tenantId}}](https://login.microsoftonline.com/%7B%7BtenantId%7D)}/oauth2/v2.0/token`
- `scope : [https://management.azure.com/.default](https://management.azure.com/.default)`

Once done, you can save and call it **Azure** or something similar.

### Lets generate our OAuth 2.0 Token

1. In Thunder Client, click 'New Request'

    ![/assets/images/your-first-api-request/new-request.png](/assets/images/your-first-api-request/new-request.png)

2. Select Auth, then head to OAuth 2.0 and ensure Grant Type is set to **Client Credentials**.

OAuth is THE standard in terms of cloud / identity. I concur that it’s rough to start with… Though do each flow via direct calls (without using an SDK) to get it “into your fingers”. Once you know what is going on, please only use an SDK, as with security… the slightest slip-up will haunt you!

Now you can input the following parameters to generate the token required for the API Request.

`Grant Type: Client Credentials`

`Token Url: { {accessTokenUrl} }`

`Client ID: { {clientId} }`

`Client Secret: { {clientSecret} }`

`Scope : { {scope} }`

![/assets/images/your-first-api-request/oauth.png](/assets/images/your-first-api-request/oauth.png)

You should now be able to generate a token!

If you interested in the breakdown of the token you can paste it into here [https://jwt.ms/](https://jwt.ms/) a JSON Web Token decoder and take a look!

### Now lets call the API

1. To find the REST API you wish to call, you can head to [https://resources.azure.com/](https://resources.azure.com/) or look take a look at the [Microsoft documentation](https://docs.microsoft.com/en-us/rest/api/resources/resources) for each API operation.
2. Next locate the API URI (Uniform Resource Idenifier) it should look something like this: `` https://management.azure.com/subscriptions/<Subscription-ID>?api-version=2014-04-01 ``
3. With that URI you can enter it into the requests box. Depending on what API you choose you can change the request function. In this example case we'll call the `GET` [List All Resources API](https://docs.microsoft.com/en-us/rest/api/resources/resources/list) - this will list all the resources within your given subscription and the data associated to each resource.
4. Set the request type to **GET** and populate the URI with the following, remember to swap out the actual subscription ID with the ``{ {subscriptionID} }`` placeholder `https://management.azure.com/subscriptions/{ {subscriptionId} }/resources`
5. Populate the query parameters with the appropriate api-version, this can change depending on the API you are calling (this can happen automatically if you copy the full request) in our case `api-version : 2021-04-01`

    ![/assets/images/your-first-api-request/api-call.png](/assets/images/your-first-api-request/api-call.png)

6. Should receive a successful **200 OK status code**. If you receive anything else, take a look a these [HTTP Status code requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).

    ![/assets/images/your-first-api-request/statuscode.png](/assets/images/your-first-api-request/statuscode.png)

### Great... Now what?

You have made your first API request! Well done. The returned response is in JSON Format, using this in conjuction with a script means you can generate all sorts of reports or filter out items with specific parameters and then make changes to them. The possibilities are endless.

Other interesting articles on REST API's and the cloud

- [https://aster.cloud/2019/09/25/how-to-call-azure-rest-apis/](https://aster.cloud/2019/09/25/how-to-call-azure-rest-apis/)
- [https://blog.jongallant.com/2021/02/azure-rest-apis-postman-2021/](https://blog.jongallant.com/2021/02/azure-rest-apis-postman-2021/)
- [https://mauridb.medium.com/calling-azure-rest-api-via-curl-eb10a06127](https://mauridb.medium.com/calling-azure-rest-api-via-curl-eb10a06127)