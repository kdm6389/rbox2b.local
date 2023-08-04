# setup ssh

```[stpi@rbox2b ~]$ ./ssh
[stpi@rbox2b .ssh]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/stpi/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/stpi/.ssh/id_rsa
Your public key has been saved in /home/stpi/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:8d42KqooIkywxlwU/8aH0x/M0wES/UrpztU3yNETZOc stpi@rbox2b
The key's randomart image is:
+---[RSA 3072]----+
|   ..    oo.  .o.|
|   ..     ... .o.|
|  .  .  .   o.. E|
|.  .  o oooo.o.o |
|+..    *So+=o.+ .|
|.=    . o..+o+ o.|
|+         +.=   o|
|+. .    .  = .   |
|o.. .... ..      |
+----[SHA256]-----+
[stpi@rbox2b .ssh]$ mv authorized_keys authorized_keys.1
[stpi@rbox2b .ssh]$ mv id_rsa.pub authorized_keys
[stpi@rbox2b .ssh]$ cat id_rsa```



id_rsa.pub ====> is public key ===> it will stay at remote machine 
is_rsa =====> private key (keep with you, from where you login) you may need to convert this to other formate as well
