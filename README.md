|Title |  Object Storage |
|-----------|----------------------------------|
|Author | Kenneth Chen |
|Utility | IBM, Softlayer, Object Storage |
|Date | 10/08/2018 |

# Part I. CLI commands

Check if you have three queries commands.
- `curl` will get files by URL  
- `jq` jason query is a query in dictionary  
- `tee` is a log output  

```
# which curl jq tee

/usr/bin/curl
/usr/bin/jq
/usr/bin/tee
```
If you have, it will show the path. `jq` is default in Ubuntu Debian. If not, install them. 
```
# sudo apt-get install jq
```
Try something that you know will result in unsuccessful file transfer so as to learn what kind of these commands function. 
```
# curl -s https://api.softlayer.com/... | tee softlayer.$(date +%s).log | jq .id
```
tee will generate `softlayer.xxxxxxxxx.log`. You can observe what the log file recorded. 

# Part II. Object Storage 

Check if you have an object storage in your softlayer account. Replace USERID:API_KEY respectively. If you have the object storage, you'd see Swift object storage. If you don't have yet, tee (softlayer.xxxxxxx.log) will generate nothing. 
```
# curl --user USERID:API_KEY https://api.softlayer.com/rest/v3.1/SoftLayer_Account/getHubNetworkStorage | tee softlayer.$(date +%s).log | jq '.[] | select(.vendorName == "Swift") | del (.properties)'  
```
#### How to order object storage by REST API,  
https://softlayer.github.io/blog/waelriac/managing-softlayer-object-storage-through-rest-apis/

To get the price item id for your softlayer account, replace USERID:API_KEY respectively. 
```
# curl -s --user USERID:API_KEY https://api.softlayer.com/rest/v3.1/SoftLayer_Product_Package/0/getItems | tee softlayer.$(date +%s).log | jq '.[] | select(.keyName == "OBJECT_STORAGE_PAY_AS_YOU_GO") | .prices[0].id'

16984
```
To order with the price item id, 
```
# curl --user USERID:API_KEY https://api.softlayer.com/rest/v3.1/SoftLayer_Product_Order/placeOrder -X POST -d '{"parameters" : [{"complexType": "SoftLayer_Container_Product_Order_Network_Storage_Hub", "quantity": 1, "packageId": 0, "prices": [{ "id": 16984}]}]}' | tee softlayer.$(date +%s).log | jq '{"orderId": .orderId, "status": .placedOrder.status, "location": .orderDetails.locationObject.name}'

{
  "orderId": 30204687,
  "status": "PENDING_AUTO_APPROVAL",
  "location": "lon05"
}
```
Checking if the object storage is complete
```
# curl --user USERID:API_KEY https://api.softlayer.com/rest/v3.1/SoftLayer_Account/getHubNetworkStorage | tee softlayer.$(date +%s).log | jq '.[] | select(.vendorName == "Swift") | del (.properties)'  

{
  "accountId": 1729689,
  "capacityGb": 0,
  "createDate": "2018-10-15T21:35:40-04:00",
  "guestId": null,
  "hardwareId": null,
  "hostId": null,
  "id": 53336359,
  "nasType": "HUB",
  "password": "",
  "serviceProviderId": 1,
  "storageTypeId": "15",
  "upgradableFlag": true,
  "username": "SLOS1729689-2",
  "serviceResourceBackendIpAddress": "https://dal05.objectstorage.service.networklayer.com/auth/v1.0/",
  "serviceResourceName": "OBJECT_STORAGE_DAL05",
  "vendorName": "Swift"
}
```
To manage the containers in the object storage, you can simply do by swift utility. 
```
# pip install python-swiftclient
# swift -A https://SL_DATA_CENTER_ID.objectstorage.softlayer.net/auth/v1.0/ -U USERNAME:ACCOUNT-ID -K API_KEY list
```
- SL_DATA_CENTER_ID = sjc01  
- USERNAME = object storage username, SLOS1729689-2
- ACCOUNT-ID = softlayer account username associated with API

real command, change the API_KEY
```
# swift -A https://sjc01.objectstorage.softlayer.net/auth/v1.0/ -U SLOS1729689-2:SL1729689 -K API_KEY list
```
# Jupyter Notebook

Since I'm going to work in jupyter notebook, I'll install the notebook.

```
# apt-get update && apt-get upgrade
# apt install python3-pip
# pip3 install jupyter
# pip3 install softlayer-object-storage
```

### Setting up Jupyter Notebook

You need to set up the configuration file for jupyter notebook to work in your browser. Remember, you're launching jupyter-notebook from the server. But you will get access to jupyter-notebook from your local machine browser. 

```
# jupyter notebook --generate-config

Writing default config to: /root/.jupyter/jupyter_notebook_config.py
```
Modifying the configuration file
```
# cd /root/.jupyter/
# vi jupyter_notebook_config.py
```
Copy the following script and paste it the jupyter_notebook_config.py
```
c = get_config()
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.port = 8123
```
Launch jupyter-notebook, you somehow need to bind with the IP
```
# jupyter-notebook --ip=50.23.91.124 --allow-root
```

Go to your browser, launch a new Python3 notebook
```
50.23.91.124:8123/?token ....
```
------------
## Utilization of Object Storage done in Jupyter Notebook


