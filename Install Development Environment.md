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

