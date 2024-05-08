# SSH Configuration
```shell
## SSH Configuration
sudo systemctl status ssh
sudo systemctl enable ssh
sudo systemctl start ssh

sudo apt update && sudo apt upgrade
sudo apt install openssh-server
sudo cp /etc/ssh/sshd_config{,.bak}
sudo nano /etc/ssh/sshd_config
sudo systemctl reload ssh
ssh remote_host
ssh remote_username@remote_host
exit
```
