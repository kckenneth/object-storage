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

## Manage the object storage by curl

### 1. To get the X-Auth-Token and X-Storage-URL (valid for 24 hours)
```
# curl -i -H "X-Auth-User: SLOS1729689-2:USERID " -H "X-Auth-Key: API_KEY " https://dal05.objectstorage.softlayer.net/auth/v1.0

HTTP/1.1 200 OK
Content-Length: 1536
X-Auth-Token-Expires: 24799
X-Auth-Token: AUTH_tk3948b783e16c4e48a3d726a76b7ff60b
X-Storage-Token: AUTH_tk3948b783e16c4e48a3d726a76b7ff60b
X-Storage-Url: https://dal05.objectstorage.softlayer.net/v1/AUTH_b7619532-8c35-4938-bf47-773773206815
Content-Type: application/json
X-Trans-Id: txa992607906754a01b6079-005bc63145
Date: Tue, 16 Oct 2018 18:43:18 GMT
```

### 2. To get the list of the container 
```
# curl -i -H "X-Auth-Token: AUTH_tk3948b783e16c4e48a3d726a76b7ff60b" https://dal05.objectstorage.softlayer.net/v1/AUTH_b7619532-8c35-4938-bf47-773773206815

HTTP/1.1 200 OK
Content-Length: 6
X-Account-Meta-Nas-Id: 53336359
X-Account-Object-Count: 0
X-Account-Storage-Policy-Standard-Container-Count: 1
X-Timestamp: 1539653747.99715
X-Account-Meta-Cdn-Id: 99655
X-Account-Storage-Policy-Standard-Object-Count: 0
X-Account-Bytes-Used: 0
X-Account-Container-Count: 1
Content-Type: text/plain; charset=utf-8
Accept-Ranges: bytes
X-Account-Storage-Policy-Standard-Bytes-Used: 0
X-Trans-Id: tx400e4333a71d44e0a7a50-005bc633ee
Date: Tue, 16 Oct 2018 18:54:38 GMT

week7
```
This list the container named 'week7'. 

### 3. Create a new container

If you want to create a new container, just use the 'list the container' command, but with `-XPUT` flag and `container name` at the end of the X-Storage-url. I'm creating another container `mybucket`. I'm using the X-Auth-Token and X-Storage-Url because they will expire after 24 hours. 
```
# curl -i -XPUT -H "X-Auth-Token: AUTH_tk3948b783e16c4e48a3d726a76b7ff60b" https://dal05.objectstorage.softlayer.net/v1/AUTH_b7619532-8c35-4938-bf47-773773206815/mybucket

HTTP/1.1 201 Created
Content-Length: 0
Content-Type: text/html; charset=UTF-8
X-Trans-Id: tx4b9ac5d7fb5c46c981861-005bc636a7
Date: Tue, 16 Oct 2018 19:06:15 GMT
```
### 4. Check again how many containers I have. 
```
# curl -i -H "X-Auth-Token: AUTH_tk3948b783e16c4e48a3d726a76b7ff60b" https://dal05.objectstorage.softlayer.net/v1/AUTH_b7619532-8c35-4938-bf47-773773206815

mybucket
week7
```
### 5. Check the content of the container 

You can check the content of the container by appending the X-Storage-Url by the container name. It's like a subfolder. 
```
# curl -i -H "X-Auth-Token: AUTH_tk3948b783e16c4e48a3d726a76b7ff60b" https://dal05.objectstorage.softlayer.net/v1/AUTH_b7619532-8c35-4938-bf47-773773206815/week7

bigrams
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

https://github.com/kckenneth/object-storage/blob/master/Object_storage.ipynb
