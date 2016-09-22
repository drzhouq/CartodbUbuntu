Cartodb Ubuntu 14.04
=============
http://cartodb.readthedocs.io/en/latest/run.html

CartoDB VM on Ubuntu Server 14.04 LTS
based upon https://gist.githubusercontent.com/tomay/7779546/raw/5479e0be07e80b1cc5b4341c180d675afff5b427/cartodb+install+notes+12.04.md

### Cartodb install on Digital Ocean ubuntu 12.04 64-bit
based on https://github.com/CartoDB/cartodb with additions as necessary
export DEBIAN_FRONTEND=noninteractive


sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8

Install git

    sudo apt-get install git-core

Clone project

    git clone --recursive https://github.com/CartoDB/cartodb.git

Some dependencies
py
    apt-get update
    sudo apt-get install python-software-properties
    **sudo apt-get install software-properties-common**

More dependencies

    sudo apt-get install unp
    sudo apt-get install zip

GDAL

    sudo apt-get install gdal-bin libgdal1-dev

GEOS

    # note: seems all ok from above; 0 upgraded, 0 newly installed, 0 to remove and 20 not upgraded.
    sudo apt-get install libgeos-c1 libgeos-dev

JSON-C

    sudo apt-get install libjson0 python-simplejson libjson0-dev

PROJ-4

    sudo apt-get install proj-bin proj-data libproj-dev

POSTGRESQL

    # Note: 12.04 so not using carto ppa here: sudo add-apt-repository ppa:cartodb/postgresql 
    ***sudo apt-get install postgresql-9.1 postgresql-client-9.1 postgresql-contrib-9.1 postgresql-server-dev-9.1
    sudo apt-get install postgresql-9.3 postgresql-client-9.3 postgresql-contrib-9.3 postgresql-server-dev-9.3

## Cartodb Postgres Extension
#this should be done right after postgreq is installed

    # version is 0.5.2 but it keeps changing ...
    git clone --branch 0.5.2 https://github.com/CartoDB/cartodb-postgresql && \
      cd cartodb-postgresql && \
      PGUSER=postgres sudo make install
      
PLP-PYTHON

    ***sudo apt-get install postgresql-plpython-9.1
    sudo apt-get install postgresql-plpython-9.3

POST-GIS    
    PostGIS 2.1 is included in 14.04
    # sudo apt-get install postgis postgresql-9.3-postgis-2.1

Configure spatial-db tempate

    # create and run setup script
    touch /var/lib/postgresql/config_db.sh

    nano /var/lib/postgresql/config_db.sh

    #!/usr/bin/env bash
    POSTGIS_SQL_PATH=/usr/share/postgresql/9.3/contrib/postgis-2.1
    createdb -E UTF8 template_postgis
    createlang -d template_postgis plpgsql
    psql -d postgres -c \ "UPDATE pg_database SET datistemplate='true' WHERE datname='template_postgis'"
    psql -d template_postgis -f $POSTGIS_SQL_PATH/postgis.sql
    psql -d template_postgis -f $POSTGIS_SQL_PATH/spatial_ref_sys.sql
    psql -d template_postgis -f $POSTGIS_SQL_PATH/legacy.sql
    psql -d template_postgis -f $POSTGIS_SQL_PATH/rtpostgis.sql
    psql -d template_postgis -f $POSTGIS_SQL_PATH/topology.sql
    psql -d template_postgis -c "GRANT ALL ON geometry_columns TO PUBLIC;"
    psql -d template_postgis -c "GRANT ALL ON spatial_ref_sys TO PUBLIC;"

    sudo su - postgres 
    bash config_db.sh

    # confirm template exists with
    psql --command="\l"
    exit

Install Ruby 1.9.2 (using RVM)

    sudo su
    \curl -L https://get.rvm.io | bash
    source /etc/profile.d/rvm.sh
    rvm install 1.9.2

Node

    sudo apt-get install nodejs npm

Redis

    sudo apt-get install redis-server


ImageMagik 

    # something to do with windshaft and comparing images
    sudo apt-get install imagemagick

Install Python dependencies

    sudo apt-get install python-setuptools
    sudo apt-get install python-dev
    sudo? easy_install pip
to make sure gdal installation, following stuff has to be exported
export CPLUS_INCLUDE_PATH=/usr/include/gdal
export C_INCLUDE_PATH=/usr/include/gdal
export PATH=$PATH:/usr/include/gdal

    sudo -E pip install --no-use-wheel -r cartodb/python_requirements.txt
alternatively, one could skip the gdal because gdal has been alreay installed
cat python_requirements.txt | grep -v gdal | sudo pip install -r /dev/stdin
    # when python-gdal bindings fail to install using above
    sudo apt-add-repository ppa:ubuntugis/ubuntugis-unstable
    # Instead, we need add the ppa
    
    sudo add-apt-repository ppa:ubuntugis/ppa
    sudo apt-get update
    sudo apt-get install python-gdal

    # leaving this ppa in for now??
    sudo apt-add-repository --remove ppa:ubuntugis/ubuntugis-unstable
    apt-get update

Varnish
    # not sure how useful is this, not in the official document, and varnish cache is not installed
    # RealGeeks version recommended in Cartodb docs
    sudo pip install -e git+https://github.com/RealGeeks/python-varnish.git@0971d6024fbb2614350853a5e0f8736ba3fb1f0d#egg=python-varnish

Mapnik
    
    # following not working, note these seem to be v 2.1.1 
    ## sudo apt-get install libmapnik-dev python-mapnik mapnik-utils

    # so remove the cartodb ppa
    sudo apt-add-repository --remove ppa:cartodb/mapnik
    
    # install mapnik ppa
    sudo add-apt-repository ppa:mapnik/nightly-2.1 
    apt-get update

    # install boost (q: with suggests? --install-suggests, a: so far seems ok without)
    sudo apt-get install libboost1.55-tools-dev

    # then install mapnik
    sudo apt-get install libmapnik-dev python-mapnik mapnik-utils

    # can test with
    python
    import mapnik

### CartoDB SQL API

    # nodejs-legacy fixes 'WARN This failure might be due to the use of legacy binary "node"' error
    sudo apt-get install nodejs-legacy
    git clone git://github.com/CartoDB/CartoDB-SQL-API.git
    cd CartoDB-SQL-API
    git checkout master
    before we can do this, we need to update npm, the default npm version is too low
    sudo npm install npm -g
    npm install

    mv config/environments/test.js.example config/environments/test.js

    # run test
    export PGUSER=postgres # note see config below
    make check # 143 tests pass
    The tests won't run unless you modify the test/support/redis_utils.js to require the helper.js under the test folder.
    reported this as an issue to cartodb

### Windshaft-cartodb

    # note about g++ memory error (only 512 MB RAM for me: http://stackoverflow.com/questions/14800609/phusion-passenger-nginx-module-installation-error-on-ubuntu-server-12-04-2-64)
    git clone git://github.com/CartoDB/Windshaft-cartodb.git
    cd Windshaft-cartodb
    git checkout master
    needs the stupid libpango
    sudo apt-get install libpango1.0-dev
    npm install

    mv config/environments/development.js.example config/environments/development.js
    # set mapnik version "2.1.1"
    # create millstone directory writable
    mkdir -p /tmp/cdb-tiler-dev/millstone-dev

    # testing
    # 1. setup test db and user


    make check


### Additional pg config

    ## pg
    #In /etc/postgresql/9.3/main/pg_hba.conf, set auth method to "trust":
    local   all             postgres                                trust
    host    all             all             127.0.0.1/32            trust

    # restart psql
    sudo service postgresql restart
    
    # set pw (also add to database.yml -- optional/redundant given above??
    sudo -u postgres psql
    \password
    (Set password) 
    \q

### CartoDB Schema Triggers
### need to check out the release tag for all the github stuff
    git clone https://github.com/CartoDB/pg_schema_triggers.git && \
      cd pg_schema_triggers && \
      sudo make all install && \
      sudo sed -i \
      "/#shared_preload/a shared_preload_libraries = 'schema_triggers.so'" \
      /etc/postgresql/9.3/main/postgresql.conf

    
### Setup (see https://github.com/CartoDB/cartodb for details)

    export SUBDOMAIN=carto # unset SUBDOMAIN to undo
    cd cartodb
  
  in order to run test (grunt test)
  The jasmine verion in package.json needs to be upgraded to 0.9
   "grunt-contrib-jasmine": "0.9.2" instead of the older version. Of course, again, this is related with whatever node version you have
  
    # bundle --> carto seems to require ruby 1.9.3 so installed this first
    rvmsudo install ruby-1.9.3
    needs ruby-2.3.1 for some of the ruby modules to be installed 1.93 is not going to work
    after this install bundler with gem
    rvmsudo gem install bundler
    rvm use 1.9.3@cartodb --create && bundle install # from cartodb/
    # sudo bundle install # not needed? anyway bundle not found for sudo
bundle exec grunt --environment development


    mv config/app_config.yml.sample config/app_config.yml
    nano config/app_config.yml
    # only change so far: developers_host:    'http://carto.localhost.lan:3000'

    mv config/database.yml.sample config/database.yml
    nano config/database.yml

    echo "127.0.0.1 ${SUBDOMAIN}.localhost.lan" | sudo tee -a /etc/hosts
    # add similar entry to local /etc/hosts
    xxx.xxx.xxx.xxx carto.localhost.lan
    xxx.xxx.xxx.xxx admin.localhost.lan
Add 
gem 'net-telnet' to Gemfile and run bundle install

RAILS_ENV=development bundle exec rake db:create
RAILS_ENV=development bundle exec rake db:migrate

### Create user script and alter limits
    # start redis-server with redis-server
    # create user script (edit to avoid error): sh script/create_dev_user ${SUBDOMAIN}
    bundle exec rake cartodb:db:set_user_quota["myuser",10240] 
    bundle exec rake cartodb:db:set_unlimited_table_quota["myuser"]
    bundle exec rake cartodb:db:set_user_private_tables_enabled["myuser",'true']
    bundle exec rake cartodb:db:set_user_account_type["myuser",'[DEDICATED]'] 

be careful the cartodb postgresql version. The facility to create dev user assume the verison was 0.1.15, but there are new versions, which does not provide the upgrade path to the older version.  

vim /home/ubuntu/cartodb/app/models/user/db_service.rb
 def upgrade_cartodb_postgres_extension(statement_timeout = nil, cdb_extension_target_version = nil)
        if cdb_extension_target_version.nil?
          #cdb_extension_target_version = '0.11.5', how come this is hardcoded
          cdb_extension_target_version = '0.17.1'



### Start all servers
    bundle exec script/resque
    be careful, it might complain UTF-8 coding is incorrect. You might have to mannually change it use something like:
    egrep "#( )?encoding: UTF-8(')?" app -rl|xargs sed -i 's/UTF-8'\?/utf-8/g'
    
    bundle exec thin start --threaded -p 3000 --threadpool-size 5
    
    # foreman not working for me - but starting services individually is ...
    Of course, we need to make sure foreman gem is actually added before the following command. Foreman just pick up the Procfile and run those for you, nothing black magic
    bundle exec foreman start -p 3000
    
### Useful miscellany
    
    apt-cache policy mapnik-utils

    # drop db
    sudo su - postgres
    dropdb carto_db_development

    # drop roles
    sudo -u postgres psql
    DROP ROLE publicuser
    DROP ROLE tileuser

    # node tests
    export PGUSER = "postgres"
    make check # 143 tests complete

    # restart pg
    sudo service postgresql restart
    
    #travis environment for Winshaft 
    {
  "dist": "trusty",
  "sudo": "required",
  "addons": {
    "postgresql": "9.5",
    "apt": {
      "packages": [
        "postgresql-plpython-9.5",
        "pkg-config",
        "libcairo2-dev",
        "libjpeg8-dev",
        "libgif-dev",
        "libpango1.0-dev"
      ]
    }
  },
  "before_install": [
    "createdb template_postgis",
    "createuser publicuser",
    "psql -c \"CREATE EXTENSION postgis\" template_postgis"
  ],
  "env": "NPROCS=1 JOBS=1 PGUSER=postgres",
  "language": "node_js",
  "node_js": "0.10",
  "group": "stable",
  "os": "linux"
}

for SQL_API
{
  "before_install": [
    "lsb_release -a",
    "sudo mv /etc/apt/sources.list.d/pgdg.list* /tmp",
    "sudo apt-get -qq purge postgis* postgresql*",
    "sudo rm -Rf /var/lib/postgresql /etc/postgresql",
    "sudo apt-add-repository --yes ppa:cartodb/postgresql-9.5",
    "sudo apt-add-repository --yes ppa:cartodb/gis",
    "sudo apt-get update",
    "sudo apt-get install -q postgresql-9.5-postgis-2.2",
    "sudo apt-get install -q postgresql-contrib-9.5",
    "sudo apt-get install -q postgresql-plpython-9.5",
    "sudo apt-get install -q postgis",
    "sudo apt-get install -q gdal-bin",
    "sudo apt-get install -q ogr2ogr2-static-bin",
    "echo -e \"local\\tall\\tall\\ttrust\\nhost\\tall\\tall\\t127.0.0.1/32\\ttrust\\nhost\\tall\\tall\\t::1/128\\ttrust\" |sudo tee /etc/postgresql/9.5/main/pg_hba.conf",
    "sudo service postgresql restart",
    "psql -c 'create database template_postgis;' -U postgres",
    "psql -c 'CREATE EXTENSION postgis;' -U postgres -d template_postgis",
    "./configure"
  ],
  "env": "PGUSER=postgres",
  "language": "node_js",
  "node_js": "0.10",
  "group": "stable",
  "dist": "precise",
  "os": "linux"
}


for carto

{
  "language": "node_js",
  "node_js": "0.10",
  "before_install": [
    "git submodule update --init --recursive"
  ],
  "install": [
    "npm install"
  ],
  "before_script": [
    "npm install -g npm@2.14",
    "npm install -g grunt-cli"
  ],
  "script": [
    "grunt test"
  ],
  "group": "stable",
  "dist": "precise",
  "os": "linux"
}


{
  "language": "node_js",
  "node_js": "0.10",
  "before_install": [
    "lsb_release -a",
    "sudo mv /etc/apt/sources.list.d/pgdg-source.list* /tmp",
    "sudo apt-get -qq purge postgis* postgresql*",
    "sudo rm -Rf /var/lib/postgresql /etc/postgresql",
    "sudo apt-add-repository --yes ppa:cartodb/postgresql-9.3",
    "sudo apt-add-repository --yes ppa:cartodb/gis",
    "sudo apt-get update",
    "sudo apt-get install gdal-bin libgdal1-dev python-gdal",
    "sudo apt-get install postgresql-9.3-postgis-2.1 postgresql-plpython-9.3",
    "sudo apt-get install postgis",
    "sudo apt-get install -q unp zip ruby1.9.3 ruby1.9.1-dev python-pip ruby-rspec libicu-dev",
    "sudo apt-get install postgresql-contrib-9.3",
    "echo -e \"local\\tall\\tall\\ttrust\\nhost\\tall\\tall\\t127.0.0.1/32\\ttrust\\nhost\\tall\\tall\\t::1/128\\ttrust\" |sudo tee /etc/postgresql/9.3/main/pg_hba.conf",
    "sudo service postgresql restart",
    "sudo su postgres -c \"createdb template_postgis\"",
    "echo \"SELECT VERSION();\" | sudo su postgres -c \"psql template_postgis\"",
    "sudo su postgres -c \"echo 'CREATE EXTENSION postgis;' | psql template_postgis\"",
    "sudo su postgres -c \"echo 'SELECT POSTGIS_FULL_VERSION();' | psql template_postgis\"",
    "sudo apt-get install postgresql-server-dev-9.3",
    "hg clone https://bitbucket.org/malloclabs/pg_schema_triggers && cd pg_schema_triggers && make && sudo make install && cd -",
    "echo \"shared_preload_libraries = 'schema_triggers.so'\" | sudo tee -a /etc/postgresql/9.3/main/postgresql.conf && sudo service postgresql restart",
    "cd lib/sql",
    "make all && sudo make install",
    "PGUSER=postgres make installcheck || cat regression.diffs",
    "cd -"
  ],
  "before_script": [
    "npm install -g grunt-cli"
  ],
  "install": [
    "gdal-config --version",
    "ruby --version",
    "sudo gem install bundler",
    "gem install debugger-linecache -v '1.1.2' -- --with-ruby-include=$rvm_path/src/ruby-1.9.3-p484/",
    "bundle install",
    "cp config/app_config.yml.testing config/app_config.yml",
    "cp config/database.yml.sample config/database.yml",
    "cat python_requirements.txt | grep -v gdal | sudo pip install -r /dev/stdin",
    "npm install"
  ],
  "script": [
    "make travis"
  ],
  "env": null,
  "group": "stable",
  "dist": "precise",
  "os": "linux"
}
