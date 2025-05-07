# FRP (Fast Reverse Proxy) Server Setup on AWS EC2 (Ubuntu)

This guide outlines the steps to configure and deploy an **FRP server** on an **Ubuntu-based EC2 instance**.

---

## 1. Prerequisites

- An EC2 instance (named **FRP Instance**) running **Ubuntu**.
- [FRP binaries](https://github.com/fatedier/frp/releases) downloaded from GitHub.
- Open port **7000** (or others as needed) in the security group.
- Basic knowledge of **Linux command-line** operations.

---

## 2. Launch an EC2 Instance (FRP Instance)

1. Sign in to the **AWS Management Console**.
2. Navigate to **EC2 > Launch Instance**.
3. Choose an **Ubuntu AMI** (e.g., Ubuntu 22.04).
4. Select an instance type (e.g., `t2.micro` for free-tier).
5. Configure instance details as needed.
6. Create or choose a **key pair** for SSH access.
7. Create or select a **security group**.
8. Click **Launch Instance**.

---

## 3. Modify Security Group to Open Port 7000

1. Go to **Security Groups** from EC2 dashboard.
2. Select the group attached to your instance.
3. Click **Inbound Rules > Edit**.
4. Add a new rule:
   - **Type**: Custom TCP
   - **Port Range**: 7000
   - **Source**: `0.0.0.0/0` or your specific IP
5. Click **Save rules**.

---

## 4. Connect to EC2 via PuTTY

### Convert PEM to PPK

1. Install **PuTTY** and **PuTTYgen**.
2. Open **PuTTYgen**, load your `.pem` file.
3. Save as `.ppk`.

### Connect Using PuTTY

1. Open **PuTTY**.
2. Set **Host Name** to EC2's **Public IP**.
3. Go to **Connection > SSH > Auth** → Load the `.ppk` file.
4. Click **Open** and log in as `ubuntu`.

---

## 5. Install FRP on EC2

```bash
sudo apt update
sudo apt install wget curl -y

wget https://github.com/fatedier/frp/releases/download/v0.47.0/frp_0.47.0_linux_amd64.tar.gz
tar -zxvf frp_0.47.0_linux_amd64.tar.gz
cd frp_0.47.0_linux_amd64

sudo mkdir -p /opt/frp
sudo mv frps frps.ini /opt/frp/
sudo chmod +x /opt/frp/frps

```

## 6. Configure the frp server

```
sudo nano /opt/frp/frps.ini
```
```
[common]
bind_port = 7000
vhost_http_port = 80
vhost_https_port = 443
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
```

## 7.Run the frp server

```
sudo /opt/frp/frps -c /opt/frp/frps.ini
```

## 8. Set Up FRP as a Systemd Service

```
sudo nano /etc/systemd/system/frps.service
```
paste

```
[Unit]
Description=FRP server
After=network.target

[Service]
ExecStart=/opt/frp/frps -c /opt/frp/frps.ini
Restart=always
User=nobody
RestartSec=3

[Install]
WantedBy=multi-user.target
```
then

check status

```
sudo systemctl daemon-reload
sudo systemctl start frps
sudo systemctl enable frps
```

## 9. Enable the FRP Dashboard (Optional)

```
http://<your-ec2-public-ip>:7500
```

## 10. Connect an FRP Client

```
wget https://github.com/fatedier/frp/releases/download/v0.47.0/frp_0.47.0_linux_amd64.tar.gz
tar -zxvf frp_0.47.0_linux_amd64.tar.gz
cd frp_0.47.0_linux_amd64
```

```
[common]
server_addr = <your-ec2-public-ip>
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

```
./frpc -c ./frpc.ini
```

## 11. SSH via FRP Tunnel

```
ssh -p 6000 user@<your-ec2-public-ip>
```
## ✅ Conclusion
Your FRP server is successfully deployed on AWS EC2! You can now perform secure port forwarding and remote access with ease 🚀
