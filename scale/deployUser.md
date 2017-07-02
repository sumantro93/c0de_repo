(wherever it says url.com, use your server's domain or IP)

Login to new server as root, then add a deploy user
```bash
sudo useradd --create-home -s /bin/bash deploy
sudo adduser deploy sudo
sudo passwd deploy
```
And Update the new password


Now login as that user 
```bash
ssh deploy@url.com
```

Make directory .ssh on the remote server and log out
```bash
mkdir .ssh
exit
```

Push your ssh key to the authorized_keys file on the remote server
```bash
scp ~/.ssh/id_rsa.pub deploy@url.com:~/.ssh/authorized_keys
```

