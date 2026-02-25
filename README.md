# How To SSH

1. **Create SSH key pair in your personal computer**. This will create your personal SSH private and public key.
```bash
ssh-keygen
```

2. **Copy your SSH public key to the remote server.**
From your personal PC run this command.
```bash
ssh-copy-id server_user@server_ip_address
```

Then you'll be prompted for the **remote server's user account password** in order to complete copying your personal SSH public key to the remote server.

Example: ssh-copy-id rockylinux_user@192:168:98:22

3. **Login to server using SSH**
```bash
ssh server_user@server_ip_address
```

Example: ssh rockylinux_user@192:168:98:22
