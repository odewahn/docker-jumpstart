# A quick intro to SSH

boot2docker uses SSH to allow you to connect to the box.  this appendix will provide some basic info for windows and mac users about how ssh works.  It will cover:

* Installing an SSH client
* About ~/.ssh 
* keypairs
  * Private key -- the unique string that identifies your computer
  * Public key -- the companion string you give to other servers you want to connect to
* authorized\_keys files

To connect to another server, your public key must be listed in that server's "known\_host" file.  