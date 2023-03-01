# Setting multiple GitHub accounts & ssh keys
Sometimes, we have multiple projects using different GitHub accounts and so, ssh keys.
When come the moment to set and use a second account / ssh key, the first one will "always" be used by default.
The solution is to configure ssh agent and then use it the right way with git commands. 

## Step 1: ssh keys
Create a ssh key:
```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
# Enter a file name (eg.: id_ed25519_work)
```
Start the user-agent before adding key:
```sh
eval "$(ssh-agent -s)"
# Agent pid 59566
```
Add the key to the user-agent:
```sh
ssh-add ~/.ssh/id_ed25519_work
```
Checking if the key is registered to the user-agent:
```sh
ssh-add -l
# 2048 6d:65:b9:3b:ff:9c:5a:54:1c:2f:6a:f7:44:03:84:3f your_email@example.com (ed25519)
```
Modifying ssh key file permissions:
```sh
# For public key: 644 (-rw-r--r--)
chmod 644 ./id_ed25519_work.pub
# For private key: 600 (-rw-------)
chmod 600 ./id_ed25519_work.pub
# Checking permissions octal level
stat -c "%a %n" *
```

## Step 2: ssh config
Set up multiple ssh profiles by creating/modifying `~/.ssh/config` file:

### Default GitHub
```
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa
```

### "Work" GitHub
```
Host work.github.com
    User workerName
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_ed25519_work
```
| Note the slightly differing `Host` values

## Step 3: test
Verifying the ssh connection with the correct keys:
```sh
ssh -T git@github.com
# Hi <userName>! You've successfully authenticated, but GitHub does not provide shell access.
ssh -T git@work.github.com
# Hi workerName! You've successfully authenticated, but GitHub does not provide shell access.
```
| Note the correspondence between git@work.github.com and the `.ssh/config` Host work.github.com

## Step 4: clone a a repository
Now it's possible to clone a repository using a specific ssh config/key.
```
git clone git@work.github.com:userName/projectName.git
```

# Similar solutions
Solutions that will maybe works when there are multiple ssh keys, but only one account.

Ssh cloning by specifying the ssh key:
```
git clone -c core.sshCommand="/usr/bin/ssh -i ~/.ssh/id_ed25519_work" git@github.com:userName/projectName.git
```
Modifying an existing project by forcing a key to be used:
```
git config --local core.sshCommand "/usr/bin/ssh -i ~/.ssh/id_ed25519_work"
```

# links
- [Create a ssh key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [Add ssh key on Github](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
- [Test ssh connection](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection)
- [Multiple GitHub Accounts & SSH Config](https://stackoverflow.com/questions/3225862/multiple-github-accounts-ssh-config)
