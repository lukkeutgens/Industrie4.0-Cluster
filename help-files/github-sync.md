# Github repository setup
For syncing files between the control plane server and Github we gonna setup github on the server

## Install GitHub
```bash
sudo apt update
sudo apt install git
```

## Generate SSH Key (recommended for secure access)
On the control plane server:
```bash
ssh-keygen -t ed25519 -C "iotadmin@csrv1"
```
The `-C` option gives an extra comment to the key to reconize it.
Enter the following when asked:
- filename: For example iotadmin-github-key
- passphrase: Add a password for more security

The public key will be saved as: `iotadmin-github-key`and `iotadmin-github-key.pub`
Make sure these files are set under `~/.ssh/` folder
Now read the content of the public key because we will need it to paste it on our github page
```bash
cat ~/.ssh/iotadmin-github-key.pub
```

## Make the SSH-config for github
```bash
vi ~/.ssh/config
```
Add content:
```bash
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/iotadmin-github
  IdentitiesOnly yes
```

## Add SSH Key to GitHub
Go to GitHub → Settings → SSH and GPG keys
Click New SSH key
Paste the public key content (iotadmin-github-key.pub)
Give it a name like ControlPlane-Industrie4.0

## Test connection
```bash
ssh -T git@github.com
```
Returns: Hi lukkeutgens! You've successfully authenticated, but GitHub does not provide shell access.
Connection is ok, don't mind the remark about shell access. Git only allows Git-operations.

---

## Clone the Repository to Your Server
Create a folder for where the repository should be cloned. In my case: `~/github/`
Move into that folder:
```bash
cd ~/github/
```
Get the repository to the local server:
```bash
git clone git@github.com:lukkeutgens/Industrie4.0-Cluster.git
```

## Check Status Before Syncing
Always check what Git sees before syncing. Use command in the local repository folder:
```bash
cd ~/gitbub/Industrie4.0-Cluster.git/
git status
```
This shows:
- Untracked files
- Modified files
- Files staged for commit

## Sync from GitHub to Server (Pull)
Use this when you've made changes online via the GitHub website or from another machine, and want to update your local copy:
```bash
cd ~/github/Industrie4.0-Cluster
git pull --rebase
```
--rebase ensures a clean history by stacking your local commits on top of the latest remote ones.

If you have local changes that aren't committed yet, Git will warn you. In that case, either commit them first or stash them:
```bash
git stash push -m "Temporary stash before pull"
git pull --rebase
git stash pop
```

## Sync from Server to GitHub (Push)
```bash
cd ~/github/Industrie4.0-Cluster
git add .
git commit -m "Updated documentation and configuration files"
git push origin main
```
If your branch is already set to track origin/main, you can just use git push.

---

## After server reboot
It could be that the ssh key needs to be reloaded after a server reboot:
```bash
ssh-add ~/.ssh/iotadmin-github
```

## Forcing sync from server to Github
**Force push with lease**:
```bash
git push --force-with-lease origin main
```
This forces the push only if the remote branch hasn’t changed since your last pull (an extra safety check).

**Brute force**:
It overwrites the remote branch (main) with your local version and all commits on GitHub that you don’t have locally will be **deleted**. 
This is powerful but risky — use it only if you're certain your local branch is the correct one.
```bash
git push --force origin main
```
