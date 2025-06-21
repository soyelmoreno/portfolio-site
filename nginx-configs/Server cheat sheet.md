# Linux Ubuntu web server command line cheat sheet

## Nginx server commands

Status, reload, restart, start, stop

    nst = sudo systemctl status nginx
    nrel = sudo systemctl reload nginx
    nsta = sudo systemctl start nginx
    nsto = sudo systemctl stop nginx
    nres = sudo systemctl restart nginx

Check syntax of config. Or verbose: actually see it

    sudo nginx -t
    sudo nginx -T

What user is Nginx web server using:

    ps aux | grep nginx

## Set permissions

View permission as numeric

    stat -c '%a %U:%G %n' *
    # This adds "directory" or "regular file"
    stat -c '%a %U:%G %F %n' *

Owner read/write, group read, others none

    sudo chmod 640 index.html

Directories: 755. Owner read/write/execute, group read/execute, others read/execute

    sudo find /var/www -type d -exec chmod 755 {} \;

Files: 644. Owner read/write, group read, others read

    sudo find /var/www -type f -exec chmod 644 {} \;

## Set ownership

Change ownership for a single file like this (username:groupname)

    sudo chown www-data:www-data index.html
    sudo chown hotshot:www-data index.html

Recursive into all files and directories

    sudo chown -R www-data:www-data /var/www/cmoreno.me
    sudo chown -R www-data:www-data /var/www/hello.cmoreno.me

# Create a new user group

    sudo groupadd www-cm
    sudo groupadd www-hellocm
    sudo groupadd www-bookcm

# Add a user to a group

    sudo usermod -aG www-cm hotshot
    sudo usermod -aG www-cm www-data

# Set group ownership and setgid

    sudo chown -R hotshot:www-cm /var/www/cmoreno.me
    sudo chmod g+s /var/www/cmoreno.me
