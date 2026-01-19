# V Server Setup

## Introduction

This is the documentation for the `V-Server-Setup` project. A virtual Ubuntu machine with root access was provided.

> Note: *Sensitive data like user and ip are omitted. They only appear in form of placeholders.*

## Table of contents

1. [Tasks](#tasks)
2. [SSH](#ssh)
    - [Access via public key](#access-via-public-key)
    - [Prohibit login with password](#prohibit-login-with-password)
3. [NGINX](#nginx)
    - [Installation](#installation)
    - [Configure alternative index.html on a custom port](#configure-alternative-indexhtml-on-a-custom-port)
4. [GIT](#git)
    - [GitHub](#github)
5. [Tests](#tests)
    - [SSH Tests](#ssh-tests)
    - [NGINX Tests](#nginx-tests)
    - [GIT Tests](#git-tests)
    
## Tasks

Setup a v-server with the following components and functionality:

- SSH
    - Access via public key
    - Prohibit login with password

- NGINX
    - Installation
    - Configure alternative index.html on a custom port

- GIT
    - Setup user and SSH key

## SSH

### Access via public key

Generate an SSH key pair on your local machine with
```sh
ssh-keygen -t ed25519 -f ~/.ssh/my_vm_key -C "my_vm_key"
```
*Use [ECC](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography) like ed25519 for security and performance.*


Confirm you have access to the v-server.
```sh
ssh user@host
```
This forces you to enter your password.

Now copy your **public** key to the server with
```sh
ssh-copy-id -i ~/.ssh/my_vm_key.pub user@host
```
This will add the key to the `authorized_keys` of your user.

Confirm you can login without a password. 
```sh
ssh -i ~/.ssh/my_vm_key user@host
```

Tip: You can set an alias for this command with
```sh
alias ssh_my_vm='ssh -i ~/.ssh/my_vm_key user@host'
```
This lets you connect to the server with `ssh_my_vm`

### Prohibit login with password

Open the ssh daemon config with
```sh
sudo nano /etc/ssh/sshd_config
```
and change the line `#PasswordAuthentication yes` to `PasswordAuthentication no`.

Then restart the daemon with
```sh
sudo systemctl restart ssh
```

Confirm login with password is denied.
```sh
ssh user@host
```

## NGINX

### Installation

Use the package manager to install the nginx package.
```sh
sudo apt install nginx
```

Check the webserver status.
```sh
systemctl status nginx
```
Nginx should be running. Otherwise start it with
```sh
sudo systemctl start nginx
```

Open your browser on your local machine and type in the host ip. You should see the default nginx welcome page.
This confirms your webserver is reachable from the outside.

### Configure alternative index.html on a custom port

Create a new directory under `/var/www/` for your alternative index file. This will also be the root in your config.
```sh
sudo mkdir /var/www/alternatives
```
*Usually .html files are placed inside `/var/www/html`. We use a different directory for clarity*

Create the `alternate-index.html` with
```sh
sudo touch /var/www/alternatives/alternate-index.html
```
You can edit this later with an editor of your choice. For example:
```sh
sudo nano /var/www/alternatives/alternate-index.html
```

Minimal example index:
```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello, Nginx</title>
</head>
<body>
    <h1>This is the alternate-index.html</h1>
</body>
</html>
```

Once your new index is edited, create a config file named `alternatives` under `/etc/nginx/sites-enabled/`
```sh
sudo nano /etc/nginx/sites-enabled/alternatives
```
with the following content:
```nginx
server {
    listen 8081;
    listen [::]:8081;

    root /var/www/alternatives;
    index alternate-index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Then restart nginx with 
```sh
sudo systemctl restart nginx
```

Nginx now serves the default index on port 80 and your alternate-index on port 8081. You can test this with your browser on your local machine.

## GIT

Once again, generate an SSH key pair, but this time on the v-server.
```sh
ssh-keygen -t ed25519 -C "vm_git_key"
```

Then print the content of your public key
```sh
cat ~/.ssh/id_ed25519.pub
```
and copy it into your clipboard.

### GitHub

Open [GitHub](https://github.com/) in your browser, then go to `Settings > SSH and GPG keys` and click the button `New SSH key`.
Set a title, preferably matching your comment. In this example `vm_git_key`.
Then paste the content of your public key into the `key` field and click the button `Add SSH key`.

## Tests

### SSH Tests

- Login with pubkey: 
```sh
ssh -i ~/.ssh/my_vm_key user@host
```

- Deny login with password:
```sh
ssh user@host
```
*This should not work.*

### NGINX Tests

- Default page is still reachable:
    Open your browser and type the ip of your server into the address bar.

- Alternative page is reachable:
    Open your browser and type the ip followed by :8081 into the address bar.

### GIT Tests

- GitHub recognizes your SSH key:
```sh
ssh -T git@github.com
```
You should see a message like `Hi user! You've succesfully....`