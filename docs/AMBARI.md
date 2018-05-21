# AMBARI
## Installation Guide for Ambari 2.6.2
https://cwiki.apache.org/confluence/display/AMBARI/Installation+Guide+for+Ambari+2.6.2

## 异常
### ambari-agent注册失败
 * 异常代码
``` 
Connecting to https://cluster01.hadoop:8440/ca
NetUtil.py:88 - [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:579)
NetUtil.py:89 - SSLError: Failed to connect. Please check openssl library versions.
Refer to: https://bugzilla.redhat.com/show_bug.cgi?id=1022468 for more details.
NetUtil.py:116 - Server at https://cluster01.hadoop:8440 is not reachable, sleeping for 10 seconds...
``` 
从字面上看为openssl版本的问题，实际是因为python版本的问题。 
在python 2.7.5及以上版本时，增加了certificate verification，正是因为这个特性导致ambari-agent无法连接server。
 * 解决方法： 
  修改/etc/python/cert-verification.cfg配置文件,然后重启ambari agent，就可以正常注册 
```
vim /etc/python/cert-verification.cfg
verify=platform_default ###(这是默认配置)
verify=disable ###(修改后)
```
 * 参考：https://blog.csdn.net/cy309173854/article/details/79487079