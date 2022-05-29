### Content

- [Debian server set up for FastAPI](#debian-server-set-up-for-fastapi)
- [Install needed packages](#install-needed-packages)
- [Create user www (Situationally)](#create-user-www-situationally)
- [Setup SSH for ssh-key authentication (Optional)](#setup-ssh-for-ssh-key-authentication-optional)
- [Install Python3.10](#install-python310)
- [Pull Git project and run server](#pull-git-project-and-run-server)
- [Hints](#hints)

# Debian server set up for FastAPI
In this guide we will set up Debian server for FastAPI project. We will install all necessary packages, build Python3.10.4 (current version at time of repository creation) from sources, and run `install.sh` script to configure nginx and gunicorn daemon under systemd.

## Install needed packages
Connect to Debian server and update repositories and install some initial needed packages:
```sh
sudo apt-get update
sudo apt-get install -y git nginx
```
## Create user www (Situationally)
if when creating server you specified a login other than `www` or you didn't have opportunity to configure a user, then you need to create a user `www`.
Create user `www`, add `www` to sudoers and login as `www`:
```sh
sudo adduser www
sudo usermod -aG sudo www
su - www
```
## Setup SSH for ssh-key authentication (Optional)
Using ssh keys is more secure than using a password for authentication.
If you don't have ssh keys, you need to generate them with following command (**command must be executed locally**).
```sh
ssh-keygen
```
To install keys on a remote server, use following command (**command must be executed locally**).
```sh
ssh-copy-id www@server_ip_address
```
Configure SSH in file `/etc/ssh/sshd_config` with settings below:
* AllowUsers www
* PermitRootLogin no
* PasswordAuthentication no

Restart SSH server to update settings:
```sh
sudo service ssh restart
```
## Install Python3.10
Build from source Python3.10, install with prefix to `~/.python` folder:
```sh
sudo apt install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev
wget https://www.python.org/ftp/python/3.10.4/Python-3.10.4.tgz
tar xvf Python-3.10.*
cd Python-3.10.4
mkdir ~/.python
./configure --enable-optimizations --prefix=/home/www/.python
make -j8
sudo make altinstall
sudo ln -s /home/www/.python/bin/python3.10 /bin/python
```
> Note: After installation, Python source folders can be deleted, they are no longer needed.

## Pull Git project and run server
>  **To quickly start server without editing configurations, you need download this repository and place FastAPI project files in `src` folder, while application file must be called `main.py` and application must be called `app`!!!
You also need append all dependencies in `requirements.txt` file.**

```sh
git pull your_repository
```
> **Before running installation script, you need to verify python installation by running `python` command.**

To start server, you need to run `install.sh` script and specify domain.
For example:
```
$ ./install.sh
Your domain: my-domain.com
```
## Hints
> **Gunicorn** logs contains in `gunicorn/access.log` and `gunicorn/error.log`.
> **Nginx** logs contains in `nginx/access.log` and `/nginx/error.log`

For check status of **gunicorn** daemon, run:
```sh
sudo systemctl status gunicorn
```
For test new **nginx** config after change, run:
```sh
sudo nginx -t
```
When changing systemd or nginx configuration, you need to restart unit using `restart.sh` script:
```sh
./restart.sh
```