SSH
===

## Weak keys ##

Regenerating weak system SSH host keys (make sense only for RSA and ECDSA):

```bash
# remove old and unused keys
sudo rm /etc/ssh/ssh_host_*key*
# create new
sudo ssh-keygen -N '' -o -t ed25519 -f /etc/ssh/ssh_host_ed25519_key
sudo ssh-keygen -N '' -o -t rsa -b 4096 -f /etc/ssh/ssh_host_rsa_key
sudo ssh-keygen -N '' -o -t ecdsa -b 521 -f /etc/ssh/ssh_host_ecdsa_key
```

Print key fingerprints (good idea would be to save this info and check by clients on the first connect):

```bash
for file in /etc/ssh/*.pub; do ssh-keygen -lf $file; done
```

Generate user keys:

```bash
ssh-keygen -o -a 256 -t ed25519
ssh-keygen -o -a 256 -t rsa -b 4096
ssh-keygen -o -a 256 -t ecdsa -b 521
```

Print key fingerprints:

```bash
for file in ~/.ssh/*.pub; do ssh-keygen -lf $file; done
```

## Configure server and enable 2FA ##

Add hardening options in `/etc/ssh/sshd_config`:

```
Port 8192
LogLevel VERBOSE
Subsystem sftp /usr/lib/openssh/sftp-server -f AUTHPRIV -l INFO
AllowUsers dp
PermitRootLogin no
PasswordAuthentication no
X11Forwarding no
AllowTcpForwarding no
AllowStreamLocalForwarding no
GatewayPorts no
PermitTunnel no
ClientAliveInterval 3600
ClientAliveCountMax 3
UsePAM yes
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
```

Setup [two-factor authentication for SSH](https://www.vultr.com/docs/how-to-setup-two-factor-authentication-2fa-for-ssh-on-debian-9-using-google-authenticator). 

Check and apply:

```bash
sshd -t && service ssh reload && service ssh restart
```


## Limit remote access ##

There are some approaches below sorted by preference.

### Host without Docker ###

The most secure and efficient approach is to use `iptables`:

```bash
# put your port and IPs here
iptables -A INPUT -p tcp --dport 8192 --source 10.0.0.10,10.0.0.100 -j ACCEPT
iptables -A INPUT -p tcp --dport 8192 -j DROP
```

Install a special service to restore `iptables` rules after reboot:

```bash
# rules will be saved during the installation
apt install iptables-persistent
# or save manually if necessary
iptables-save > /etc/iptables/rules.v4
```

### Host with Docker ###

It's difficult to use `iptables` in this case as Docker injects its own rules.
So it's necessary to configure which hosts can connect using TCP wrappers.
By default, deny all hosts in file `/etc/hosts.deny`:

```
sshd: ALL
```

Then list allowed hosts in `/etc/hosts.allow`:

```
sshd: 10.0.0.10, 10.0.0.100
```

### Native SSH approach ###

List all allowed IPs in `/etc/ssh/sshd_config` file:

```
AllowUsers = *@10.0.0.10 *@10.0.0.100
``` 


## Moduli ## 

The `/etc/ssh/moduli` file contains prime numbers and generators for use by sshd in the Diffie-Hellman Group Exchange key exchange method.
The [Diffi-Hellman key exchange](http://en.wikipedia.org/wiki/Diffie-Hellman_key_exchange) is used in the beginning of SSH sessions to generate a shared secret between the client and the server.

```bash
cd /tmp

# generate (can take a couple of days)
for b in 2048 3072 4096 5120 6144 7168 8192; do 
    echo "$b"
    # initial candidate generation pass
    ssh-keygen -G moduli.$b.gen -b $b
    # second primality testing pass
    ssh-keygen -T moduli.$b.out -f moduli.$b.gen
    # append
    cat moduli.$b.out >> moduli
    # clear temp files
    rm moduli.$b.gen
    rm moduli.$b.out
done

# replace system file
mv ./moduli /etc/ssh/moduli
```


## Useful links ##

* [Secure Secure Shell](https://stribika.github.io/2015/01/04/secure-secure-shell.html)
* [Upgrade your SSH keys!](https://blog.g3rt.nl/upgrade-your-ssh-keys.html)
* [Setup two-factor authentication for SSH](https://www.vultr.com/docs/how-to-setup-two-factor-authentication-2fa-for-ssh-on-debian-9-using-google-authenticator).
