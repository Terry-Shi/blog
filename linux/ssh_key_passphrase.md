#### ssh key passphrase works in windows but not in linux
#### PROBLEM:  
ssh with key `HPIT.ppk` is working at windowns (ssh client is Mobaxterm, also working with Putty)  
command: `ssh -i ~/Desktop/HPIT.ppk hos@16.202.70.65`

But failed to login in linux with same key file:
keep showing msg: `Enter passphrase for key 'HPIT.ppk':`

Try for more detail output with command: `ssh -v -i HPIT.ppk hos@16.202.70.65`  v means verbose.  
found below msg:
```
debug1: Next authentication method: publickey
debug1: Trying private key: HPIT.ppk
debug1: key_parse_private2: missing begin marker
debug1: key_parse_private_pem: PEM_read_PrivateKey failed
debug1: read PEM private key done: type <unknown>
Enter passphrase for key 'HPIT.ppk':
```
Obviously, the format of key file is not right.

#### ANSWER:  
The Linux SSH client (typically OpenSSH) can't read the PPK format used by the Windows SSH client Putty. You need to convert the "PPK" key given to you into an OpenSSH key first. Install "putty" on Linux and use the puttygen command line tool:
```
$ sudo aptitude install putty
$ mkdir -p ~/.ssh
$ puttygen ~/HPIT.ppk -o ~/.ssh/id_rsa -O private-openssh
```
Enter your passphrase, and you'll get an OpenSSH-compatible key in the standard location ~/.ssh/id_rsa. Afterwards you can just use ssh-add(without any arguments!) to add this key to the SSH agent.

Alternatively you can use the PUTTYgen program provided by putty on Windows.
load the key file and go to "Conversions > Export OpenSSH Key", then just saved it as <keyname>.pem

#### Ref:
- [ppk key sample](HPIT.ppk)
- [pem key sample](HPIT.pem)
- http://stackoverflow.com/questions/9631246/ssh-key-passphrase-works-in-windows-but-not-in-linux