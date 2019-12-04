---
title:  "Lessons learnt from deploying Azure Search via CI-CD"
excerpt: "Learn about how you can overcome some limitations when deploying Azure Search Service indexes via your CI/CD automation pipeline."
header:
 image: https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2019/12/azure-search-index.png
categories: 
  - Tech
tags:
  - azure
  - deployment
  - search
  - json
---

As we move more towards the cloud for our solutions these days, there are a growing number of PAAS options appearing that can help us to solve our problems without needing to reinvent the wheel.

If your application requires a search service to be able to quickly query over a proprietary or legacy data source, I can highly recommend the use of Azure Search. Azure Search offers many features, including fast setup, data source flexibility, great REST API and extensive monitoring and control via the Azure Portal. You can read more about [Azure Search here](https://docs.microsoft.com/en-us/azure/search/).

This blog post, however, is about how to cater for possible issues when deploying an Azure Search and its corresponding indexes via a CI/CD DevOps pipeline.

The best way to deploy the search service itself is via ARM templates. The ARM template just sets up the search service, it can’t be used to populate the indexes. Below is a sample template which is all you need to get started.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "searchServiceName": {
      "defaultValue": "MUST BE PROVIDED",
      "type": "string"
    },
    "searchServiceTier": {
      "defaultValue": "standard",
      "type": "string"
    },
    "searchServiceLocation": {
      "defaultValue": "Australia East",
      "type": "string"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Search/searchServices",
      "sku": {
        "name": "[parameters('searchServiceTier')]"
      },
      "name": "[parameters('searchServiceName')]",
      "apiVersion": "2015-08-19",
      "location": "[parameters('searchServiceLocation')]",
      "scale": null,
      "properties": {
        "replicaCount": 1,
        "partitionCount": 1,
        "hostingMode": "default"
      },
      "dependsOn": []
    }
  ],
  "outputs": {}
}
```

To create your search indexes, you can either use the Azure Portal or you can create/update JSON files. These files can be sent as updates to the Search Service via the REST API or an SDK as part of an automated deployment pipeline.

The Azure Portal is a good way to get started. You can design your index and ensure all the correct properties are there.

![Azure Search Index UI](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2019/12/azure-search-index.png)

In this modern era, it is preferable to have the indexes either created or exported from the web portal as JSON files. These files should be part of a git repository which is linked to build and deploy pipelines. When it gets to the deployment pipeline, the files will be picked up and applied to the search service.

With automated deployments, pushing any index changes out to the various environments a painless process. Also storing the JSON files in a git repository will maintain change histories and allow for concurrent development of the index definitions.

```json
{
  "name": "people",
  "fields": [
    {
      "name": "id",
      "type": "Edm.String",
      "facetable": false,
      "filterable": false,
      "key": true,
      "retrievable": true,
      "searchable": false,
      "sortable": false
    },
    {
      "name": "name",
      "type": "Edm.String",
      "facetable": false,
      "filterable": true,
      "key": false,
      "retrievable": true,
      "searchable": true,
      "sortable": true
    },
    {
      "name": "address",
      "type": "Edm.String",
      "facetable": false,
      "filterable": true,
      "key": false,
      "retrievable": true,
      "searchable": true,
      "sortable": true
    },
    {
      "name": "nationality",
      "type": "Edm.String",
      "facetable": true,
      "filterable": true,
      "key": false,
      "retrievable": true,
      "searchable": true,
      "sortable": true
    },
    {
      "name": "dateofbirth",
      "type": "Edm.DateTimeOffset",
      "facetable": true,
      "filterable": true,
      "key": false,
      "retrievable": true,
      "searchable": false,
      "sortable": true
    }
  ],
  "suggesters": [
    {
      "name": "nameSuggester",
      "sourceFields": [
        "name"
      ]
    }
  ],
  "scoringProfiles": [],
  "defaultScoringProfile": "",
  "corsOptions": null
}
```

A standard Azure Search Service allows for up to 50 indexes. If your application is searching through large amounts of data, it may need all 50. However if you try to create up to 50 indexes at once as part of a deployment pipeline, you will end up with an error like:

> Error with creating index people.json. The remote server returned an error: (503) Server Unavailable. Detailed error: You are sending too many requests. Please try again later.

Unfortunately, Azure Search only lets you create about 25 indexes in a 2-minute window. This means that to reach the allowed 50, you have to make 25 indexes, wait 2 minutes and then create another 25.

To bypass this issue, your deployment script will need to take into account these restrictions and handle a delay with the process. I have created a PowerShell script that is [defined here](https://github.com/platformeng/powershell-scripts/blob/master/Azure/Search/New-SearchIndexes.ps1) that does exactly that.

If you use the script or some variation of it, in your deployment pipeline, it will pause after deploying 20 indexes and resume again 2 minutes later. This will repeat until all indexes have been deployed.

![PowerShell script output sample](https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2019/12/2019-10-01_14-31-10.png)

So why is this limitation there in the first place? I have asked Microsoft Support and *(as of the date this blog was published)* there has been no reasonable response.

I assume that they expected users to only use the Azure Portal for manually creating indexes, as opposed to an automated process. This is a bit disappointing as a standard search service can have up to 50 indexes at once. Having to wait up to about 6 minutes just to deploy indexes is not ideal.

Hopefully, this will be fixed soon, but until that happens the solution detailed here is ready for people to use if required.