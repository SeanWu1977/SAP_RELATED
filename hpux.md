```
# 設定目錄下新檔案的權限
setacl -m d:g:sapsys:rwx /tmp/acltest

# 設定此目錄的權限
setacl -m g:sapsys:rwx /tmp/acltest
```
