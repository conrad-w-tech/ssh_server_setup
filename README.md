SSH Remote Server Setup
Solution for roadmap.sh SSH Remote Server Setup project.
Step 1 — Provision the Remote Server
Spin up a VPS with any provider (DigitalOcean, Hetzner, AWS, etc.) running a Linux distro of your choice — Debian, Ubuntu, Fedora, Rocky, etc.
Note the server's public IP address and make sure you have initial access (root password or provider-injected key).
Step 2 — Install and Enable OpenSSH Server
SSH into the server (or use the provider's console) and install the SSH daemon:
bash# Debian/Ubuntu
sudo apt update && sudo apt install -y openssh-server

# Fedora/Rocky/RHEL
sudo dnf install -y openssh-server
Enable and start the service:
bashsudo systemctl enable --now sshd
Verify it's running:
bashsudo systemctl status sshd
Step 3 — Generate Two SSH Key Pairs (Local Machine)
On your local machine, generate two separate key pairs:
bashssh-keygen -t ed25519 -f ~/.ssh/roadmap_key1 -C "roadmap-key1"
ssh-keygen -t ed25519 -f ~/.ssh/roadmap_key2 -C "roadmap-key2"
For each key you will be prompted to set a passphrase — do it for security.
This produces:
FilePurpose~/.ssh/roadmap_key1Private key #1 — never share~/.ssh/roadmap_key1.pubPublic key #1~/.ssh/roadmap_key2Private key #2 — never share~/.ssh/roadmap_key2.pubPublic key #2
Step 4 — Copy Public Keys to the Remote Server
Use ssh-copy-id to add each public key to the server's ~/.ssh/authorized_keys:
bashssh-copy-id -i ~/.ssh/roadmap_key1.pub user@server-ip
ssh-copy-id -i ~/.ssh/roadmap_key2.pub user@server-ip
Each time you'll authenticate with the server password. After successful copy you'll see a confirmation that the key was added.
Verify both keys work:
bashssh -i ~/.ssh/roadmap_key1 user@server-ip
ssh -i ~/.ssh/roadmap_key2 user@server-ip
Step 5 — Configure SSH Aliases (Local Machine)
Create or edit ~/.ssh/config on your local machine:
Host server-key1
    HostName <server-ip>
    User <user>
    IdentityFile ~/.ssh/roadmap_key1
    IdentitiesOnly yes

Host server-key2
    HostName <server-ip>
    User <user>
    IdentityFile ~/.ssh/roadmap_key2
    IdentitiesOnly yes
Now you can connect with just:
bashssh server-key1
ssh server-key2
Step 6 — Disable Password Authentication
On the remote server, edit the SSH daemon config:
bashsudo vi /etc/ssh/sshd_config
Set the following:
PasswordAuthentication no
PubkeyAuthentication yes
Restart the SSH service:
bashsudo systemctl restart sshd

⚠️ Important: Do NOT close your current session before testing. Open a second terminal and confirm you can still log in with your keys. If something is misconfigured, you'll lock yourself out.

Stretch Goal — Install fail2ban
Install and enable fail2ban to protect against brute-force attacks:
bash# Debian/Ubuntu
sudo apt install -y fail2ban

# Fedora/Rocky/RHEL
sudo dnf install -y fail2ban
bashsudo systemctl enable --now fail2ban
The default configuration already monitors SSH and bans IPs after repeated failed login attempts. You can check the status with:
bashsudo fail2ban-client status sshd
