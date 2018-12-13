# Linux Server Configuration

### Step 1: Start a new Ubuntu Linux server 

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
- Once you are login into the site, click `Create instance`. 
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.

**Reference**
 [Amazon Lightsail](https://serverpilot.io/community/articles/how-to-create-a-server-on-amazon-lightsail.html).

### Step 2: SSH into the server

- From the `Account` menu on Amazon Lightsail, click on `SSH keys` tab and download the Default Private Key.
- Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `lightsail_key.rsa`.
- In your terminal, type: `chmod 600 ~/.ssh/lightsail_key.rsa`.
- To connect to the instance via the terminal: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@35.183.68.62`,

## Secure the server

### Step 3: Update and upgrade installed packages

```
sudo apt-get update
sudo apt-get upgrade
```

### Step 4: Change the SSH port from 22 to 2200

- Edit the `/etc/ssh/sshd_config` file: `sudo nano /etc/ssh/sshd_config`.
- Change the port number on line 5 from `22` to `2200`.
- Restart SSH: `sudo service ssh restart`.

### Step 5: Configure the Uncomplicated Firewall (UFW)

- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

  ```
  sudo ufw status                 
  sudo ufw default deny incoming  
  sudo ufw default allow outgoing 
  sudo ufw allow 2200/tcp          
  sudo ufw allow www              
  sudo ufw allow 123/udp           
  sudo ufw deny 22                
  ```

- Turn UFW on: `sudo ufw enable`. 

- Click on the `Manage` option of the Amazon Lightsail Instance, 
  then the `Networking` tab, and then change the firewall configuration to match the internal firewall settings above.

- Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

- From your local terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@35.183.68.62`



## Give `grader` access

### Step 6: Create a new user account named `grader`

- While logged in as `ubuntu`, add user: `sudo adduser grader`. 
- Enter with a password (twice) and fill out information for this new user.

### Step 7: Give `grader` the permission to sudo

- Edits the sudoers file: `sudo visudo`.

- Search for the line that looks like this:

  ```
  root    ALL=(ALL:ALL) ALL
  ```

- Below this line, add a new line to give sudo privileges to `grader` user.

  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Save and exit

### Step 8: Create an SSH key pair for `grader` using the `ssh-keygen` tool

- On the local machine:
  - Run `ssh-keygen`
  - Enter file in which to save the key  in the local directory `~/.ssh`
  - Enter in without passphrase twice. Two files will be generated (  `~/.ssh/keytest` and `~/.ssh/keytest.pub`) (I use keytest as filename)
  - Run `cat ~/.ssh/keytest.pub` and copy the contents of the file
  - Log in to the grader's virtual machine
- On the grader's virtual machine:
  - Create a new directory called `~/.ssh` (`mkdir .ssh`)
  - Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
  - Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - Restart SSH: `sudo service ssh restart`
- On the local machine, run: `ssh -i ~/.ssh/keytest -p 2200 grader@35.183.68.62`.



## Prepare to deploy the project

### Step 9: Configure the local timezone to UTC

- While logged in as `grader`, configure the time zone: `sudo dpkg-reconfigure tzdata`. 

### Step 10: Install git

- While logged in as `grader`, install `git`: `sudo apt-get install git`.

## Deploy the Item Catalog project

### Step 11: Clone and setup the Item Catalog project from the GitHub repository 

- While logged in as `grader`,  create `/var/www/catalog/` at root directory.

- Change to that directory and clone the catalog project:
  `sudo git clone https://github.com/xu1jia2qi3/item-Catalog-Application.git catalog`.

- Change to the `/var/www/catalog/catalog` directory.

- Rename the `application.py` file to `__init__.py` using: `mv application.py __init__.py`.

- In `__init__.py`, replace:

  ```
  # app.run(host="0.0.0.0", port=8000, debug=True)
  app.run(host="0.0.0.0", port=80, debug=True)
  ```

### Step 12: Authenticate login through Google

- Go to [Google Cloud Plateform](https://console.cloud.google.com/).
- Click `APIs & services` on left menu.
- Click `Credentials`.
- modify **item-Catalog-Application** project file and add http://35.183.68.62.xip.io as authorized JavaScript origins.
- Add http://35.183.68.62.xip.io/login and http://35.183.68.62.xip.io/gconnect as authorized redirect URI.
- Download the corresponding JSON file, open it et copy the contents.
- Open `/var/www/catalog/catalog/client_secret.json` and paste the previous contents into the this file.

### Step 13: Install the virtual environment and dependencies

- While logged in as `grader`, install pip: `sudo apt-get install python3-pip`.

- Install the virtual environment: `sudo apt-get install python-virtualenv`

- Change to the `/var/www/catalog/catalog/` directory.

- Create the virtual environment: `sudo virtualenv -p python3 venv3`.

- Change the ownership to `grader` with: `sudo chown -R grader:grader venv3/`.

- Activate the new environment: `. venv3/bin/activate`.

- Install the following dependencies:

  ```
  pip install httplib2
  pip install requests
  pip install --upgrade oauth2client
  pip install sqlalchemy
  pip install flask
  sudo apt-get install libpq-dev
  pip install psycopg2
  ```

### Step 14.6: Launch the Web Application

- run flask application by python by command `nohup sudo venv3/bin/python __init__.py`
- Open your browser to http://35.183.68.62 or http://35.183.68.62.xip.io (if you want to login, you have to use http://35.183.68.62.xip.io as address)

