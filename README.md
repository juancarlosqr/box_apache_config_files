# box_apache_config_files

Use this with [scotch box](https://box.scotch.io/)

## Install

- Install `virtualbox` and `vagrant`
- Clone [scotch box](https://box.scotch.io/) with this:

```
git clone https://github.com/scotch-io/scotch-box.git new-project
```

## Up

Run `cd new-project` and `vagrant up`

## Update php/mysql

- Update [mysql](https://github.com/scotch-io/scotch-box/issues/137)
- Update [php](https://github.com/maxpou/scotch-box/blob/master/pimpMyBox/php7.md) along with this:

```
sudo apt-get install -y php7.0 php7.0-fpm php7.0-mysql php7.0-pgsql php-redis php-curl libapache2-mod-php7.0 php7.0-mbstring php7.0-mcrypt php7.0-xml php7.0-zip htop
```

## Domains + SSL

- Install [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater)
- Clone this repo inside `new-project`
- Paste this into the Vagrantfile:

```
    config.hostsupdater.aliases = ["local.website.com"]

    config.vm.provider :virtualbox do |p|
      # p.gui = true
      p.customize ["modifyvm", :id, "--memory", "2048"]
      p.customize ["modifyvm", :id, "--cpuexecutioncap", "80"]
      p.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/vagrant", "1"]
    end

    config.vm.provision "shell", inline: <<-SHELL

      ## ------------------------------------------------------------------------------------ ##
      ## FIRST PART (run once and comment)                                                    ##
      ## ------------------------------------------------------------------------------------ ##

      echo "Adding composer to PATH"
      export PATH="~/.composer/vendor/bin:$PATH"

      echo "Activate SSL"
      sudo a2enmod ssl
      sudo service apache2 restart
      sudo mkdir /etc/apache2/ssl

      echo "Copy SSL files"
      sudo cp /var/www/box_apache_config_files/apache.crt /etc/apache2/ssl/apache.crt
      sudo cp /var/www/box_apache_config_files/apache.key /etc/apache2/ssl/apache.key
      sudo cp /var/www/box_apache_config_files/vhost-custom-ssl.conf /etc/apache2/sites-available/vhost-custom-ssl.conf

      ## ------------------------------------------------------------------------------------ ##
      ## SECOND PART (run each time a new domain is added)                                    ##
      ## ------------------------------------------------------------------------------------ ##

      ## SET DOMAINS (separated by blank spaces)
      DOMAINS=("local.website.com")

      ## Loop through all sites
      for ((i=0; i < ${#DOMAINS[@]}; i++)); do

        ## Current Domain
        DOMAIN=${DOMAINS[$i]}

        ## Comment if directory is already created
        echo "Creating directory for $DOMAIN..."
        sudo mkdir -p /var/www/public/$DOMAIN

        echo "Disabling $DOMAIN. Will probably tell you to restart Apache..."
        sudo a2dissite $DOMAIN.conf

        echo "So let's restart apache..."
        sudo service apache2 restart

        echo "Removing vhost config for $DOMAIN..."
        sudo rm /etc/apache2/sites-available/$DOMAIN.conf

        echo "Creating vhost config for $DOMAIN..."
        sudo cp /etc/apache2/sites-available/vhost-custom-ssl.conf /etc/apache2/sites-available/$DOMAIN.conf

        echo "Updating vhost config for $DOMAIN..."
        sudo sed -i s,scotchbox.local,$DOMAIN,g /etc/apache2/sites-available/$DOMAIN.conf
        sudo sed -i s,/var/www/public,/var/www/public/$DOMAIN,g /etc/apache2/sites-available/$DOMAIN.conf

        echo "Enabling $DOMAIN. Will probably tell you to restart Apache..."
        sudo a2ensite $DOMAIN.conf

        echo "So let's restart apache..."
        sudo service apache2 restart

      done
    SHELL
```

- Change / add domain(s) - default `local.website.com`
- `vagrant provision`

## DB Import

- Create DB
- Run inside box:

```
mysql -u root [database] < /var/www/box_apache_config_files/database_dump.sql
```

## Enjoy

## Sources

- [Scotch Box 2.0 - Our Dead-Simple Vagrant LAMP Stack Improved](https://scotch.io/bar-talk/announcing-scotch-box-2-0-our-dead-simple-vagrant-lamp-stack-improved)
- [How To Create a SSL Certificate on Apache for Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-create-a-ssl-certificate-on-apache-for-ubuntu-14-04)
- [Add self-signed port 443 vhost configuration issue](https://github.com/scotch-io/scotch-box/issues/187)
- [https://github.com/maxpou/scotch-box](https://github.com/maxpou/scotch-box)
- [Example Uses of Sed](https://www.lifewire.com/example-uses-of-sed-2201058)
