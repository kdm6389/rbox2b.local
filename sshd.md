# setup ssh
    
    [stpi@rbox2b ~]$ ./ssh
    [stpi@rbox2b .ssh]$ ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/stpi/.ssh/id_rsa):
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /home/stpi/.ssh/id_rsa
    Your public key has been saved in /home/stpi/.ssh/id_rsa.pub
    The key fingerprint is:
    SHA256:2d31AoooIkywxlwU/8aH0x/M0wES/Urpztr3yNETZOc szzz@ydoc3z
    The key's randomart image is:
    +---[RSA 3072]----+
    |   ..    oo.  .o.|
    |   ..     ... .o.|
    |  .  .  E.   o.. |
    |.  .o   oooo.o.o |
    |+..    *So+=o.+ .|
    |.=    . o..+o+ o.|
    |+         +.=   o|
    |+. .    .  = .   |
    |o.. .... ..      |
    +----[SHA256]-----+
    [stpi@rbox2b .ssh]$ mv authorized_keys authorized_keys.1
    [stpi@rbox2b .ssh]$ mv id_rsa.pub authorized_keys
    [stpi@rbox2b .ssh]$ cat id_rsa
    [stpi@rbox2b .ssh]$ cp id_rsa /boot/id_rsa_$(uname -n) #boot is fat32 you can copy it on windows also

#### id_rsa.pub ====> is public key ===> it will stay at remote machine 
#### is_rsa =====> private key (keep with you, from where you login) you may need to convert this to other formate as well



`$ MaxStartups 2:30:10` or `MaxStartups 10:30:60` for stopping multiple attck 
