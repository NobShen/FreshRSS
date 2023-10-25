# Install PHP on Ubunt 22.04
https:/tecadmin.net/how-to-install-php-on-ubuntu-22-04/
```bash
>sudo apt update && sudo apt upgrade -y
>sudo apt install software-properties-common ca-certificates lsb-release apt-transport-https
>LC_ALL=C.UTF-8 sudo add-apt-repository ppa:ondrej/php
>sudo apt update
>sudo apt install php8.2
>sudo apt install php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl
```
To uninstall php simply:
```bash
>sudo apt remove php8.2
>sudo apt remove php8.2-*
```
# Install Xdebug
```bash
>sudo apt install php-xdebug
>php -v
>sudo apt remove --auto-remove php-xdebug
>sudo apt purge php-xdebug
```
If running php -v shows Xdebug then it's installed.
Then find out where the ini file for xdebug is located.
```bash
>php --ini
```
The output may show something like /etc/php/8.2/cli/conf.d/20-xdebug.ini.  Then open that file to add:
```
zend_extension="xdebug.so"
xdebug.mode=debug
```
# Now install VS Code
https://code.visualstudio.com/docs/setup/linux
```
>sudo apt install wget gpg
>wget -O- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
>sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
>sudo sh -c 'echo "deb [arch=amd64, arm64, armhf signed-by=/etc/apt/sources.list.d/vscode.list'
>rm -f packages.microsoft.gpg
>sudo apt install apt-transport-https
>sudo apt update
>sudo apt install code
```
In VS Code, install PHP Debug extension.
Then install git: sudo apt install git
Once git is installed, clone a repository:
```
>cd ~/Downloads
>git clone https://github.com/NobShen/FreshRSS.git
```
Now open VS Code by >code
Then open the folder where the repository is located.  Open folder ~/Downloads/FreshRSS
At the top level, create launch.json file.
In the "configurations" [ section, the port should show 9003 as the connection port for xdebug.
Now check if Xdebug can single step through a simple php program.  So create a new php file with the content below:
```
<?php

/* echo is a print command */
echo "Hello World!\n";
echo "Hello again!\n";

?>
```
Put a breakpoint at the first statement and press Run and Debug to make sure 



