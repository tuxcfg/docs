SSH
===

Regenerating weak system SSH host keys (make sense only for RSA and ECDSA):

```bash
sudo ssh-keygen -N '' -t rsa -b 4096 -f /etc/ssh/ssh_host_rsa_key
sudo ssh-keygen -N '' -t ecdsa -b 521 -f /etc/ssh/ssh_host_ecdsa_key
```

Generate user keys:

```bash
ssh-keygen -N '' -t rsa -b 4096
ssh-keygen -N '' -t ecdsa -b 521
```

Print key fingerprints:

```bash
# system
for file in /etc/ssh/*.pub; do ssh-keygen -lf $file; done
# user
for file in ~/.ssh/*.pub; do ssh-keygen -lf $file; done
```
