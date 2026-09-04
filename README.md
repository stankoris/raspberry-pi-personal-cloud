# Raspberry Pi Personal Cloud with Tailscale and SFTPGo

A simple private cloud built on a Raspberry Pi.

The setup uses:

- Raspberry Pi OS Lite
- SSH
- Tailscale
- SFTPGo
- Tailscale Serve

## Architecture

```text
Laptop / Phone
      |
      | Tailscale
      | HTTPS
      v
Raspberry Pi
   pi-cloud
      |
      v
Tailscale Serve
      |
      v
SFTPGo :8080
      |
      v
/srv/cloud
```

The final SFTPGo WebClient is available only to devices connected to the same Tailscale network.

---

# 1. Install Raspberry Pi OS

## Install Raspberry Pi Imager

### Windows

1. Download **Raspberry Pi Imager** from the official Raspberry Pi website:
   https://www.raspberrypi.com/software/
2. Run the downloaded installer.
3. Start Raspberry Pi Imager.

### Linux

Download the Linux AppImage from:

https://www.raspberrypi.com/software/

Make it executable:

```bash
chmod +x Raspberry_Pi_Imager-*.AppImage
```

Run it:

```bash
sudo ./Raspberry_Pi_Imager-*.AppImage
```

## Configure the Raspberry Pi

Insert the microSD card and open Raspberry Pi Imager.

Select:

```text
Device:
Raspberry Pi 3
```

For the operating system select:

```text
Raspberry Pi OS (other)
    -> Raspberry Pi OS (Legacy, 64-bit) Lite
```

> This guide was tested with **Raspberry Pi OS (Legacy, 64-bit) Lite** on a Raspberry Pi 3B. The current Raspberry Pi OS Lite (64-bit) image caused SSH/user setup issues during this build, so the Legacy image is used here for reproducibility.

<img width="860" height="605" alt="Raspberry Pi OS selection" src="https://github.com/user-attachments/assets/2668bdba-9b00-4761-96e4-fda76c83f5d1" />

Select the microSD card as the storage device.

In **OS Customisation**, configure:

<img width="681" height="230" alt="image" src="https://github.com/user-attachments/assets/8038e36a-de01-4943-a40d-90b87545be9c" />

```text
Hostname: pi-cloud
Username: cloudadmin
Password: <strong-password>
Wi-Fi SSID: <your-wifi-name>
Wi-Fi password: <your-wifi-password>
Wireless LAN country: <your-country>
```

<img width="676" height="822" alt="image" src="https://github.com/user-attachments/assets/c4aaa3be-1736-42b9-9652-886810442e65" />

```text
Enable SSH: Yes
Authentication: Password authentication
```

<img width="675" height="811" alt="image" src="https://github.com/user-attachments/assets/6a4853b7-06db-487f-b180-ef9e167f1499" />

Write the image to the microSD card.

Insert the card into the Raspberry Pi and power it on.

---

# 2. Connect to the Raspberry Pi with SSH

From your computer:

```bash
ssh cloudadmin@pi-cloud.local
```

If `.local` hostname resolution is unavailable, use the Raspberry Pi LAN IP address:

```bash
ssh cloudadmin@<PI_LAN_IP>
```

---

# 3. Update Raspberry Pi OS

Update the package index:

```bash
sudo apt update
```

Install available package updates:

```bash
sudo apt upgrade -y
```

---

# 4. Install Tailscale

Install Tailscale:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Connect the Raspberry Pi to your Tailscale network:

```bash
sudo tailscale up
```

Tailscale will display an authentication URL.

Open the URL in a browser and authenticate the Raspberry Pi.

Install Tailscale on the laptop, phone, or other devices that will access the cloud and sign in to the same Tailscale account.

---

# 5. Verify Tailscale

On the Raspberry Pi, display its Tailscale IPv4 address:

```bash
tailscale ip -4
```

Check the Tailnet status:

```bash
tailscale status
```

From another device connected to the same Tailnet:

```bash
ping <PI_TAILSCALE_IP>
```

Test SSH through Tailscale:

```bash
ssh cloudadmin@<PI_TAILSCALE_IP>
```

Example:

```bash
ssh cloudadmin@100.80.20.15
```

At this point, the Raspberry Pi can be administered remotely through Tailscale.

```text
Laptop
   |
   | Tailscale
   v
pi-cloud
   |
   `-- SSH
```

---

# 6. Install SFTPGo

This guide uses **SFTPGo 2.7.5 ARM64**.

Download the package:

```bash
wget https://github.com/drakkan/sftpgo/releases/download/v2.7.5/sftpgo_2.7.5-1_arm64.deb
```

Install it:

```bash
sudo apt install ./sftpgo_2.7.5-1_arm64.deb -y
```

Enable and start the service:

```bash
sudo systemctl enable --now sftpgo
```

Verify the service:

```bash
systemctl status sftpgo --no-pager
```

The service should show:

```text
active (running)
```

---

# 7. Create the Cloud Storage Directory

Create the storage directory:

```bash
sudo mkdir -p /srv/cloud
```

Set SFTPGo as the owner:

```bash
sudo chown sftpgo:sftpgo /srv/cloud
```

Set directory permissions:

```bash
sudo chmod 750 /srv/cloud
```

---

# 8. Configure SFTPGo

Get the Raspberry Pi Tailscale IP:

```bash
tailscale ip -4
```

Open the SFTPGo WebAdmin interface:

```text
http://<PI_TAILSCALE_IP>:8080/web/admin
```

Create the initial SFTPGo administrator account.

Then go to:

```text
Users
-> Add
```

Create a user:

```text
Username: cloud
Password: <strong-password>
```

<img width="1561" height="777" alt="Raspberry Pi Imager OS Customisation" src="https://github.com/user-attachments/assets/ac746128-4f74-4d3b-9326-6227f6b0d330" />

Set:

```text
Storage: Local disk
Root directory: /srv/cloud
```

<img width="1561" height="777" alt="SFTPGo root directory configuration" src="https://github.com/user-attachments/assets/9dbbecfd-391a-4962-b521-0c4dd3bc9400" />

Under **ACLs**, set:

```text
Permissions: *
```

Leave **Per-directory permissions** empty.

Save the user.

---

# 9. Test the SFTPGo WebClient

Open:

```text
http://<PI_TAILSCALE_IP>:8080/web/client
```

Log in with the SFTPGo user:

```text
Username: cloud
Password: <cloud-user-password>
```

Test:

- Create a folder
- Upload a file
- Download a file
- Delete a file

---

# 10. Enable HTTPS with Tailscale Serve

On the Raspberry Pi:

```bash
sudo tailscale serve --bg http://127.0.0.1:8080
```

The first time this command runs, Tailscale may display a URL that must be opened to enable Serve for the Tailnet.

After enabling it, Tailscale will display a private HTTPS address similar to:

```text
https://pi-cloud.example.ts.net/
```

Check the Serve configuration:

```bash
tailscale serve status
```

Open the SFTPGo WebClient through HTTPS:

```text
https://pi-cloud.example.ts.net/web/client
```

Replace the example domain with the domain displayed by `tailscale serve`.

---

# 11. Verify the Setup After Reboot

Reboot the Raspberry Pi:

```bash
sudo reboot
```

After it starts again, open:

```text
https://pi-cloud.example.ts.net/web/client
```

Log in and verify that file upload and download still work.

You can also check Tailscale Serve:

```bash
tailscale serve status
```

---

# Final Architecture

```text
Laptop / Phone
      |
      | Private Tailscale network
      | HTTPS
      v
pi-cloud.tailnet.ts.net
      |
      v
Tailscale Serve
      |
      | reverse proxy
      v
127.0.0.1:8080
      |
      v
SFTPGo
      |
      v
/srv/cloud
```

The Raspberry Pi now provides private file upload and download from any device connected to the same Tailscale network.

# What I Should Know

After completing this project, I should be able to:

- Install and configure Raspberry Pi OS for headless administration.
- Connect to the Raspberry Pi using SSH.
- Add a Linux device to a Tailscale network.
- Access the Raspberry Pi remotely using its Tailscale IP.
- Run SFTPGo as a systemd service.
- Use `/srv/cloud` as the storage directory for SFTPGo.
- Configure an SFTPGo user and filesystem permissions.
- Use Tailscale Serve to expose the SFTPGo WebClient privately over HTTPS.
- Verify that the complete setup survives a Raspberry Pi reboot.
