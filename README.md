# CAP-PowerBI-Crawler

## Components
### cap-pbi-crawler
* An Azure Container Instance that browse PowerBI  dashboards and generate/upload dashboard data (permissions and related reports) to storage account
* Using generated data to compare with previous-run data to find changes, send to Teams channel if any.
* This container is short lived program that terminates itself when finished.
* Automatically pull image `pbicrawler:latest` from ACR (azure container registry)

### Service principal and service account
* Service principal  to pull image from ACR (azure container registry)
  * ACRPull role in ACR
  * Secret stored in key vault (check arm-template)
* Service account which has access to PowerBI workspace
  * Password stored in key vault (check arm-template)

### Storage account
* Storage account that hosts azure function, data generated by crawler, queue for pbix file change, and mounted volume in file shares.
* Storage account account key stored in key vault (check arm-template)

## Setup
* Create github token (or use existing ones) for ACR to create webhooks for every commit/push to master.
* Disable the automatic generated webhook for pull request since we always build and push the latest image.
* Create a ACR task using acr-tasks.yml to build, push, and purge images.
```
az acr task create --name pbicrawlercicd -f acr-tasks.yml -r 'cmrcapwus2acr' -c 'https://github.com/namvu24/pbi-crawler.git' --git-access-token <github-token> --subscription <az-subscription>
```
* Deploy ACI using arm-template

## Notes:
* The only ways to start/stop ACI are to use Azure CLI or REST API. AzureRM or Az module do not support this.
* Azure Functions doesn't support Puppeeter.
