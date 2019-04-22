System
======

## Root user ##

Move root user home directory to `/home` partition:

```bash
# correct users system config
sed -i 's/root:x:0:0:root:\/root:\/bin\/bash/root:x:0:0:root:\/home\/root:\/bin\/bash/g' /etc/passwd
# only the first time
mv /root /home && ln -s /home/root /root
# to restore link after OS reinstall
rm -rf /root && ln -s /home/root /root
```
