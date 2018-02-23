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
    sudo apt-get dist-upgrade
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
    sudo ufw status

    # Default to deny incoming connections and allow outgoing ones
    sudo ufw default deny incoming
    sudo ufw default allow outgoing

    # Allow the specified connections
    sudo ufw allow 2200/tcp
    sudo ufw allow www
    sudo ufw allow ntp

    # Enable the UFW
    sudo ufw enable
    ```
 - Check UFW status after updates `sudo ufw status` This will show that UFW is now active with the settings below:
   ```
   To                         Action      From
   --                         ------      ----
   22                         ALLOW       Anywhere
   2200/tcp                   ALLOW       Anywhere
   80/tcp                     ALLOW       Anywhere
   123/udp                    ALLOW       Anywhere
   22 (v6)                    ALLOW       Anywhere (v6)
   2200/tcp (v6)              ALLOW       Anywhere (v6)
   80/tcp (v6)                ALLOW       Anywhere (v6)
   123/udp (v6)               ALLOW       Anywhere (v6)
   ```
## 5. Add a new user
 - As root, type `sudo adduser grader`. Then follow prompts to add a password and name for this new account. 
 - Confirm addition of the new user by typing `sudo cat /etc/passwd`, the user grader should be listed in the output.
 
 ## 6. Set-up Key-based Authentication/Public Key Encryption
 - Using ssh-keygen on your local machine, create a key for the user grader. <br>
    ```
    # Create the keys using an email as comment
    ssh-keygen -C grader@udacity.com
    ```
   Follow the prompts to provide a name for this key and use the default key location (~/.ssh). <br>
   This process will create two keys on your local machine, the file with extension .pub is the public key to be transferred to the server.

## 7. Add and authenticate with Public Key
 - Login to the server as the new grader user
 - Run these commands in your home directory 
    ```
    mkdir .ssh
    touch .ssh/authorized_keys

    # Change both, directory and file, permissions to the grader user
    chmod 700 /home/grader/.ssh
    chmod 644 /home/grader/.ssh/authorized_keys

    # Change both, directory and file, ownership to the grader user
    chown grader /home/grader/.ssh
    chown grader /home/grader/.ssh/authorized_keys
    ```
 - Copy-and-paste the contents of the .pub key file created on your local machine 
   above to the server as the contents of the authorized_keys file and run `service ssh restart`
   
## 8. Disable Password Based Login and Force Login using Key Pair and disable root user login
 - On the server, logged in as the grader user, edit the sshd_config file `sudo nano /etc/ssh/sshd_config` 
 - Change the line with Password Authentication from yes to no
 - This is read only when the service starts, so to restart the ssh service `sudo service ssh restart`
 - On the server, logged in as root, edit the sshd_config file `sudo nano /etc/ssh/sshd_config`
 - Change to `PermitRootLogin no`
 - Enable grader user remote ssh login `sudo nano /etc/ssh/sshd_config` and add `AllowUsers grader`
 - Restart the ssh service `sudo service ssh restart`

## 9. Configure local Timezone to UTC
 - Start the configuration process `dpkg-reconfigure tzdata`
 - In the window that appears, use the arrow keys to Scroll to the bottom of 
   the Continents list and select `Etc or None of the Above` and then in the second list, select UTC <br>
 - Confirm time change by typing `date` on the command line <br>
 - Reference Documentation: http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442 
 
