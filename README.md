# Twitter-TI-Feeds

# In KQL Using at times we might need the TI Feed from Twitter.

Here is an azure function developped in KQL which can be used to fetched IOCs from Twitter

Query

```
externaldata(TIThread:string)[h@"https://raw.githubusercontent.com/samikroy/Twitter-TI-Feeds/main/twittertifeeds.txt"
]
with(format="txt")
| extend TIThread = todynamic(TIThread)
| project TIThread['author_id'],TIThread['created_at'],TIThread['id'],TIThread['malicious_urls'],TIThread['name'],TIThread['text'],TIThread['username']

```

Now, while we can use this query in our KQL queries and then it will also be useful to have this as a deployable template.

Here is the code for ARM Template

```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-08-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "workspaceName": {
      "type": "String"
    },
    "location": {
      "type": "String"
    }
  },
  "resources": [
    {
      "type": "Microsoft.OperationalInsights/workspaces",
      "apiVersion": "2017-03-15-preview",
      "name": "[parameters('workspaceName')]",
      "location": "[parameters('location')]",
      "resources": [
        {
          "type": "savedSearches",
          "apiVersion": "2020-08-01",
          "name": "TwitterTIFeed",
          "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
          ],
          "properties": {
            "etag": "*",
            "displayName": "TwitterTIFeed",
            "category": "Security",
            "FunctionAlias": "TwitterTIFeed",
            "query": "externaldata(TIThread:string)[ h@'https://raw.githubusercontent.com/samikroy/Twitter-TI-Feeds/main/twittertifeeds.txt']with(format='txt')| extend TIThread =  todynamic(TIThread)| project TIThread['author_id'],TIThread['created_at'],TIThread['id'],TIThread['malicious_urls'],TIThread['name'],TIThread['text'],TIThread['username']",
            "version": 1
          }
        }
      ]
    }
  ]
}
```

And you can easily deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsamikroy%2FTwitter-TI-Feeds%2Fmain%2FtwitterTISource.json)

