 ## The following steps will guide you through installing Redmine from sources.

1. Install the pre-requisites for Redmine and all its packages.


    sudo apt install gcc build-essential zlib1g zlib1g-dev zlibc ruby-zip libssl-dev libyaml-dev \
    libcurl4-openssl-dev ruby gem libapache2-mod-passenger apache2 apache2-dev libapr1-dev \
    libxslt1-dev checkinstall libxml2-dev ruby-dev vim libmagickwand-dev imagemagick sudo rails**

2. Choose a directory where to install Redmine. In this example /opt used. You can use another location, but you will need to update the following steps as necessary based on your choice.


    Install Redmine in /opt
    cd /opt
    mkdir redmine
    cd redmine
3. Get Redmine - 

    
    wget http://www.redmine.org/releases/redmine-3.4.11.tar.gz


4. Copy the file from config/database.yml to the /opt/redmine/redmine-3.4.11/config/database.yml file with the following contents…
   



    Ex: 
        production:
        adapter: postgresql
        database: redmine
        host: localhost
        username: redmine
        password: your_password
    Note that the spacing is important in this file. Under the “Production” line, each other line must be indented by two spaces, not tabs. Replace your_password with the password specified above. Remember to save. Keep in mind Postgresql passwords can't start with @ character (or other non alpha numerics).

5. Next, set up the database schema and load the initial database.

    
    cd /opt/redmine/redmine-3.4.11/config/
    bundle install
    bundle exec rake generate_secret_token
    RAILS_ENV=production bundle exec rake db:migrate
    RAILS_ENV=production bundle exec rake redmine:load_default_data

6. Do a quick test to verify that redmine is working using webrick.

    bundle exec ruby /usr/bin/rails server -b your_ip webrick -e production
Now try to connect via browser to http://your_ip:3000. Webrick is not for production systems. It is a good way to check things before getting started with Apache, though.

7. Next, let’s set up Apache.


    cd /opt/
    sudo chown -R www-data:www-data /opt/redmine
    cd /opt/redmine/redmine-3.4.11
    sudo chmod -R 755 files log tmp public/plugin_assets
    sudo chown www-data:www-data Gemfile.lock
7.1 Create a symbolic link which points from the Apache working directory to the Redmine public folder


    sudo ln -s /opt/redmine/redmine-3.4.11/public/ /var/www/html/redmine
7.2 Create a new vhost configuration

    sudo nano /etc/apache2/sites-available/master.conf
    
    and paste in:

    <VirtualHost *:80>

    ServerAdmin admin@example.com
    Servername hostname
    DocumentRoot /var/www/html/

    <Location /redmine>
    RailsEnv production
    RackBaseURI /redmine
    Options -MultiViews
    </Location>

    </VirtualHost>

###The above configuration files are placed in below path of the REPO

    path: Configurations/master.conf

Then, run:

    sudo a2dissite 000-default.conf
    sudo a2ensite master.conf

7.3 Add this file Configurations/passengers.conf from repo to /etc/apache2/mods-available/passenger.conf
    
    Restart the Apache web server:
    sudo service apache2 restart