Install a minimal Ubuntu 22.04

(1) Disable lid behavior:
>sudo nano /etc/systemd/logind.conf
HandleLidSwitch=ignore
>sudo systemctl restart systemd-logind

(2) Install ssh server:
>sudo apt-get install openssh-server
>sudo systemctl status sshd

(3) Install git:
>sudo apt install git

(4) Install apache2:
```
>sudo apt install apache2
>sudo a2enmod headers expires rewrite ssl
>sudo service apache2 stop
>sudo apt purge apache2 apache2-utils apache2.2-bin apache2-common
>sudo apt autoremove
>whereis apache2
>sudo rm -rf /etc/apach2
```
(5) Install FreshRSS:

Next, change to the install directory and download FreshRSS using git.  Do it from NobShen branch.  Make sure to sync with main branch first before clone.
>cd /usr/share/
>sudo git clone https://github.com/NobShen/FreshRSS.git

(6) Configure FreshRSS by setting the permissions so that your Web server can access the files
>cd FreshRSS
>sudo ./cli/access-permissions.sh
>sudo chmod -R g+w .

(7) Connect FreshRSS with apache2 by symlink the public folder to the root of your web directory
>sudo ln -s /usr/share/FreshRSS/p /var/www/html/

(8) Configure Apache by creating a file in /etc/apache2/sites-available:
>sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/freshrss.conf

Modify freshrss.conf to the following (see https://freshrss.github.io/FreshRSS/en/admins/10_ServerConfig.html)
=====
<VirtualHost *:80>
	DocumentRoot /var/www/html/

	#Default site...

	ErrorLog ${APACHE_LOG_DIR}/error.default.log
	CustomLog ${APACHE_LOG_DIR}/access.default.log vhost_combined
</VirtualHost>

<VirtualHost *:80>
	ServerName rss.example.net
	DocumentRoot /usr/share/FreshRSS/p/

	<Directory /usr/share/FreshRSS/p>
		AllowOverride AuthConfig FileInfo Indexes Limit
		Require all granted
	</Directory>

	ErrorLog ${APACHE_LOG_DIR}/freshrss_error.log
	# Consider piping the logs for cleaning passwords; cf. comment higher up.
	CustomLog ${APACHE_LOG_DIR}/freshrss_access.log combined

	AllowEncodedSlashes On
</VirtualHost>
=====

Once you’re done, create a symbolic link from this file to the sites-enabled folder:
>sudo ln -s /etc/apache2/sites-available/freshrss.conf /etc/apache2/sites-enabled/freshrss.conf

(9)  Install PHP and the necessary modules
>sudo apt install php php-curl php-gmp php-intl php-mbstring php-sqlite3 php-xml php-zip

Install the PHP module for Apache
>sudo apt install libapache2-mod-php

(10) Finally, restart the web server
>service apache2 restart

Now you can point your browser to FreshRSS
http://localhost/p
And follow the steps to configure FreshRSS - use sqlite3 as database since we didn't install mySQL earlier

(11) Notes:
The configuration of your instance in ./data/config.custom.php before the install process, or in ./data/config.php after the install process;
the default configuration for new users in ./data/config-user.custom.php, or in ./data/users/*/config.php for existing users;
the default set of feeds for new users in ./data/opml.xml.
Make sure to expose only the ./p/ folder to the Web, as the other directories contain personal and sensitive data.
You can create users using the cli.

If you wish to allow updates from the web interface, also give group write permissions to this folder.

# HOW TO ENABLE API ACCESS

First make sure you have created an API user.
```bash
cd /usr/share/FreshRSS

./cli/create-user.php --user username [ --password 'password' --api_password 'api_password' --language en --email user@example.net --token 'longRandomString' --no_default_feeds --purge_after_months 3 --feed_min_articles_default 50 --feed_ttl_default 3600 --since_hours_posts_per_rss 168 --max_posts_per_rss 400 ]
# --language can be: 'en' (default), 'fr', or one of the [supported languages](../app/i18n/)

./cli/update-user.php --user username [ ... ]
# Same options as create-user.php, except --no_default_feeds which is only available for create-user.php
```

#So for example:
```
cd /usr/share/FreshRSS
./cli/create-user.php --user rss --password '5876' --api_password '5678'
```
After a user with an api_password is created, you can create a hash of the user:password as follow:
```
echo -n rss:5678 | md5sum
55a3f889d4317f57603834b3d97b818c -
```
Use this hash value for subsequent API calls.  If you get "auth":1 then you're authenticated.
```
curl -X POST -d "api_key=55a3f889d4317f57603834b3d97b818c" http://192.168.254.133/p/api/fever.php
{"api_version":3,"auth":1,"last_refreshed_on_time":0}
```

# Enable Fever API for client
# FreshRSS - Fever API extension

This FreshRSS extension allows you to access FreshRSS with RSS readers that support the Fever API.

## Installation

To use it, make sure the ```fever.php``` file is located at the FreshRSS location `/p/api/fever.php` on your server and enable API access in FreshRSS.

### RSS clients

There are many RSS clients existing, and they all seem to understand the Fever API a bit differently. 
If your favorite client doesn't work properly with this API, create an issue and I will have a look. 

### Authentication

There is a drawback when using this plugin, which is the username and password combination that you have to use.
You have to set the API password as lowercase md5 sum from the string 'username:password' (this limitation exists, because the Fever API was designed in a way which is incompatible with the authentication system used by FreshRSS).

So if you use **kevin** as username and **freshrss** as password you have to set your FreshRSS API password to **4a6911fb47a87a77f4de285f4fac856d** (it MUST be lowercased, some tools seem to create it in uppercase which will not work!).

You can create the hash like this:
```bash
$ echo -n kevin:freshrss | md5sum
4a6911fb47a87a77f4de285f4fac856d
```
In your favorite RSS reader you configure **fever.php** as endpoint, **kevin** as username and **freshrss** as password.  

## Compatibility

Tested with:

- PHP 7.1
- FreshRSS 1.8.1-dev
- iOS apps: Fiery Feeds, Unread

## Features
Following features are implemented:

- fetching categories
- fetching feeds
- fetching RSS items (new, favorites, unread, by_id, by_feed, by_category, since)
- fetching favicons
- setting read marker for item(s)
- setting starred marker for item(s)
- **hot** is not supported as there is nothing in FreshRSS that is similar
- setting read marker for feed
- setting read marker for category

## Testing and error search
If this API doesn't work as expected in your RSS reader, please test it manually with the followin curl commands:
```
curl -X POST http://192.168.254.133/p/api/fever.php
```
It should give you the following result:
```
{"api_version": 3, "auth": 0}
```
Great, the base setup seems to work!

Now lets try an authenticated call, so add a body to your POST request encoded as `form-data` and one key named `api_key` with the value `your-password-hash`, as follow:
```
curl -X POST http://192.168.254.133/p/api/fever.php -d "api_key=a47c65c12018b8f8a549f24522e25f3b"
```
If you see the following then you're all good to go.
```
{"api_version": 3, "auth": 1, "last_refreshed_on_time": "1520013061"}
```
Perfect, you are authenticated and can now start testing the more advanced features. Therefor change the URL and append the possible API actions to your request parameters. Check the [original Fever documentation](https://feedafever.com/api) for more infos. 

Some basic calls are:

- http://your-freshrss-url/api/fever.php?api&items
- http://your-freshrss-url/api/fever.php?api&feeds
- http://your-freshrss-url/api/fever.php?api&groups
- http://your-freshrss-url/api/fever.php?api&unread_item_ids
- http://your-freshrss-url/api/fever.php?api&saved_item_ids
- http://your-freshrss-url/api/fever.php?api&items&since_id=some_id
- http://your-freshrss-url/api/fever.php?api&items&max_id=some_id
- http://your-freshrss-url/api/fever.php?api&mark=item&as=read&id=some_id
- http://your-freshrss-url/api/fever.php?api&mark=item&as=unread&id=some_id

Replace `some_id` with a real ID from your `freshrss_username_entry` database.

### Debugging

If nothing helps and your clients still misbehaves, add these lines to the start of `fever.api``

```php
file_put_contents(__DIR__ . '/fever.log', $_SERVER['HTTP_USER_AGENT'] . ': ' . json_encode($_REQUEST) . PHP_EOL, FILE_APPEND);
```

Then use your RSS client to query the API and afterwards check the file `fever.log`.

## About FreshRSS
[FreshRSS](https://freshrss.org/) is a great self-hosted RSS Reader written in PHP, which is can also be found here at [GitHub](https://github.com/FreshRSS/FreshRSS).

More extensions can be found at [FreshRSS/Extensions](https://github.com/FreshRSS/Extensions).

## Changelog

* Initial version

## Credits and license

This plugin was inspired by the [tinytinyrss-fever-plugin](https://github.com/dasmurphy/tinytinyrss-fever-plugin).
Thanks to @dasmurphy for sharing it!

The original plugin was released under GNU GPL version 2. I have no clue what that means for this plugin ... 
Install yt-dlp to download:

sudo curl -L https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -o /usr/local/bin/yt-dlp
sudo chmod a+rx /usr/local/bin/yt-dlp  # Make executable
sudo wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O /usr/local/bin/yt-dlp
sudo chmod a+rx /usr/local/bin/yt-dlp  # Make executable
sudo aria2c https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp --dir /usr/local/bin -o yt-dlp
sudo chmod a+rx /usr/local/bin/yt-dlp  # Make executable
To update, run:

sudo yt-dlp -U
