# Firewall Ports

A firewall is a security barrier hardware or software between your system and the Internet. It inspects and filters incoming and outgoing traffic, blocking unauthorized access while permitting legitimate communication.


## Built-In Firewalls Across Operating Systems

### Debian & Ubuntu (UFW)

Debian and Ubuntu include the Uncomplicated Firewall (UFW), which is **disabled by default**—permitting all traffic until activated.


### Red Hat & Fedora (firewalld)

Red Hat and Fedora rely on **firewalld**, also off by default. Install or enable it with YUM or DNF if it’s missing.


Both UFW and firewalld serve as front ends to **iptables**, the underlying Linux utility managing packet filtering.


### Windows & macOS

* Windows Firewall comes **enabled by default**.
* macOS Firewall is **disabled by default** but can be activated in System Preferences.



## Understanding Ports

A port is a logical communication endpoint—think of it as a “door” or “window” in your network. Each service listens on a specific port number.

When you visit websites:

* HTTP traffic → port **80**
* HTTPS traffic → port **443**

Other services use different ports. Expose only what’s necessary.

| Port | Service | Description             |
| ---- | ------- | ----------------------- |
| 22   | SSH     | Secure shell access     |
| 25   | SMTP    | Email delivery          |
| 53   | DNS     | Domain name resolution  |
| 80   | HTTP    | Unencrypted web traffic |
| 443  | HTTPS   | Encrypted web traffic   |


**Best practice:** On public web servers, open only ports **80** and **443**, or restrict other ports via IP allow-lists.

## Managing UFW on Debian/Ubuntu

1. Allow SSH first to prevent lockout:
   ```bash  theme={null}
   sudo ufw allow 22/tcp
   ```
2. Enable UFW:
   ```bash  theme={null}
   sudo ufw enable
   ```
3. Open HTTP and HTTPS:
   ```bash  theme={null}
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   ```
4. Reload to apply:
   ```bash  theme={null}
   sudo ufw reload
   ```
5. View rules with indices:
   ```bash  theme={null}
   sudo ufw status numbered
   ```
6. Delete a specific rule:
   ```bash  theme={null}
   sudo ufw delete <rule-number>
   ```

## Managing firewalld on Red Hat/Fedora

1. Install (if needed):
   ```bash  theme={null}
   sudo yum update && sudo yum install firewalld
   ```
2. Start and enable at boot:
   ```bash  theme={null}
   sudo systemctl start firewalld
   sudo systemctl enable firewalld
   ```
3. Open port permanently (e.g., HTTP):
   ```bash  theme={null}
   sudo firewall-cmd --permanent --add-port=80/tcp
   sudo firewall-cmd --reload
   ```
4. Remove a port:
   ```bash  theme={null}
   sudo firewall-cmd --permanent --remove-port=80/tcp
   sudo firewall-cmd --reload
   ```
5. Check active zones and ports:
   ```bash  theme={null}
   sudo firewall-cmd --list-all
   ```

## Inspecting Open Ports with netstat

`netstat` lists active connections and listening ports. Install it if missing:

```bash  theme={null}
# Debian/Ubuntu
sudo apt update
sudo apt install net-tools

# Red Hat/Fedora/CentOS
sudo yum install net-tools
```

Run:

```bash  theme={null}
sudo netstat -nltup
```

Sample output:

```bash  theme={null}
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1042/sshd
tcp6       0      0 :::22                   :::*                    LISTEN      1042/sshd
udp        0      0 127.0.0.1:323           0.0.0.0:*                          634/chronyd
udp6       0      0 :::323                  :::*                               634/chronyd
```

`netstat` helps you verify which ports your services are actively listening on essential for troubleshooting connectivity and firewall configurations.

## Links and References

* [UFW Documentation](https://help.ubuntu.com/community/UFW)
* [firewalld Guide](https://firewalld.org/documentation/)
* [iptables Tutorial](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page)
