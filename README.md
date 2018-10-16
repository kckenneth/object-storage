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
