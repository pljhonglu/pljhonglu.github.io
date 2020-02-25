# 给 www 用户生成密钥

```
sudo -u www ssh-keygen -t rsa -C "xxxx@xxx.xxx"
Generating public/private rsa key pair.
Enter file in which to save the key (/var/lib/nginx/.ssh/id_rsa): 
Could not stat /var/lib/nginx/.ssh: Permission denied
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
open /var/lib/nginx/.ssh/id_rsa failed: Permission denied.
Saving the key failed: /var/lib/nginx/.ssh/id_rsa.
```


