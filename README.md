# Linux_Server
Udacity Fullstack Web Development Nanodegree Program - Project 8: Linux Server

### Check it out
Instance Name: udacity-jessie-dowding <br>
IP Address: 52.10.188.173 <br>
Public URL: http://52.10.188.173/

## 1. Login and create your server
 - Go to https://lightsail.aws.amazon.com to create an aws account
 - Create a LightSail instance with Ubuntu
 - Create a instance plan, the $5 a month plan should be enough
 - Create an instance host name
 
## 2. Update all currently installed packages by running:

    ```
    apt-get update
    apt-get upgrade
    ```
## 3. Change the **SSH port** from *22* to *2200*

    ```
    # Open the SSH configuration file
    nano /etc/ssh/sshd_config

    # Change this line:
    # Port 22
    # To this:
    # Port 2200

    # Save and exit

    # Restart the service
    service ssh restart
    ```

## 4. Configure the **Uncomplicated Firewall (UFW)** to allow incoming connections for *SSH (2200)*, *HTTP (80)* and *NTP(123)*

    ```
    # Check the current status of the UFW
    ufw status

    # Default to deny incoming connections and allow outgoing ones
    ufw default deny incoming
    ufw default allow outgoing

    # Allow the specified connections
    ufw allow 2200/tcp
    ufw allow www
    ufw allow ntp

    # Enable the UFW
    ufw enable
    ```
## 5. Add a new user
 - As root, type `sudo adduser grader`. Then follow prompts to add a password and name for this new account. 
 - Confirm addition of the new user by typing `sudo cat /etc/passwd`, the user grader should be listed in the output.
 
 ## 6. Set-up Key-based Authentication/Public Key Encryption
 - Using ssh-keygen on your local machine, create a key for the user grader. <br>
   Follow the prompts to provide a name for this key and use the default key location (~/.ssh). <br>
   This process will create two keys on your local machine, the file with extension .pub is the public key to be transferred to the server.

## 7. Add Public Key to Server (EC2 instance)
 - Login to the server as the new grader user
 - Run these commands in your home directory 
    ```
    mkdir .ssh
    touch .ssh/authorized_keys
    ``` 
 - Copy-and-paste the contents of the .pub key file created on your local machine 
   above to the server as the contents of the authorized_keys file.
