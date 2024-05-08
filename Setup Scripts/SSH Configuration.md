# SSH Configuration

## 1 - Update and Install SSH

```shell
sudo apt update && sudo apt upgrade
sudo apt install openssh-server
```

## 2 - Backup SSH Configuration

```shell
sudo cp /etc/ssh/sshd_config{,.bak}
```

## 3 - Edit SSH Configuration

```shell
sudo nano /etc/ssh/sshd_config
```

change the following lines:
```conf
Port 22
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
SyslogFacility AUTH
LogLevel INFO
LoginGraceTime 120
PermitRootLogin yes
StrictModes yes
X11Forwarding yes
X11DisplayOffset 10
```

## 4 - Reload SSH

```shell
sudo systemctl reload ssh
```

## 5 - Start SSH

```shell
sudo systemctl status ssh
sudo systemctl enable ssh
sudo systemctl start ssh
```

## 6 - Connect to Remote Host

```shell
ssh remote_host
ssh remote_username@remote_host
exit
```

- note: `remote_host` is the IP address of the remote host.
- note: `remote_username` is the username of the remote host.
