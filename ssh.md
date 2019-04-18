SSH
===

Regenerating weak system SSH host keys (make sense only for RSA and ECDSA):

```bash
sudo ssh-keygen -N '' -o -t ed25519 -f /etc/ssh/ssh_host_ed25519_key
sudo ssh-keygen -N '' -o -t rsa -b 4096 -f /etc/ssh/ssh_host_rsa_key
sudo ssh-keygen -N '' -o -t ecdsa -b 512 -f /etc/ssh/ssh_host_ecdsa_key
```

Generate user keys:

```bash
ssh-keygen -o -a 256 -t ed25519
ssh-keygen -o -a 256 -t rsa -b 4096
ssh-keygen -o -a 256 -t ecdsa -b 512
```

Print key fingerprints:

```bash
# system
for file in /etc/ssh/*.pub; do ssh-keygen -lf $file; done

# user
for file in ~/.ssh/*.pub; do ssh-keygen -lf $file; done
```

The `/etc/ssh/moduli` file contains prime numbers and generators for use by sshd in the Diffie-Hellman Group Exchange key exchange method.
The [Diffi-Hellman key exchange](http://en.wikipedia.org/wiki/Diffie-Hellman_key_exchange) is used in the beginning of SSH sessions to generate a shared secret between the client and the server.

```bash
cd /tmp

# generate (can take a couple of days)
for b in 4096 5120 6144 7168 8192; do 
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

Useful links:

* [Secure Secure Shell](https://stribika.github.io/2015/01/04/secure-secure-shell.html)
* [Upgrade your SSH keys!](https://blog.g3rt.nl/upgrade-your-ssh-keys.html)
