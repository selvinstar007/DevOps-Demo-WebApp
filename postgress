#!/bin/bash -ex

# This script is for use with the DevOps Challenge of installing PostgreSQL 9.6 on to a   instance running Ubuntu.

# Section 1 - Variable Creation

# $rfolder is the install directory for PostgreSQL
rfolder='/postgres'
# $dfolder is the root directory for various types of read-only data files
dfolder='/postgres/data'
# $sysuser is the system user for running PostgreSQL
sysuser='postgres'
# $scripts directory
scripts="/home/$USER/scripts"
# $sqlscript is the sql script for creating the PSQL user and creating a database.
sqlscript="$scripts/initial-table.sql"
# $logfile is the log file for this installation.
logfile='psqlinstall-log'

# Section 2 - Package Installation

# Ensures the server is up to date before proceeding.
echo "Updating server..."
sudo apt-get update -y >> $logfile

# This for-loop will pull all packages from the package array and install them using apt-get
echo "Installing PostgreSQL dependencies"
sudo apt-get install ${packages[@]} -y >> $logfile


# Section 3 - Create required directories
echo "Creating folders $dfolder..."
sudo mkdir -p $dfolder >> $logfile
sudo mkdir -p $scripts >> $logfile

# Section 4 - Create system user

#echo "Creating system user '$sysuser'"
#sudo adduser --system $sysuser >> $logfile

# Section 5 - Installing  PSQL

echo "installing PostgreSQL"
sudo apt-get install postgresql postgresql-contrib 
echo "host    all             all             0.0.0.0/0               md5" >> /etc/postgresql/10/main/pg_hba.conf
sed -i 's+localhost+*+gI' /etc/postgresql/10/main/postgresql.conf
sed -i 's+#listen_addresses+listen_addresses+gI' /etc/postgresql/10/main/postgresql.conf
#listen_addresses = 'localhost'
# Section 7 - Start PSQL

echo "Wait for PostgreSQL to finish starting up..."
sleep 5


# Section 8 - initial-table.sql script is ran

# The initial-table.sql script is downloaded and ran to create the user, database, and populate the database.
echo "Downloading SQL script"
wget -P $scripts https://raw.githubusercontent.com/devopsbc01/Scripts/master/initial-table.sql

echo "Running script"
#$rfolder/bin/psql -U postgres -f $sqlscript

sudo -u postgres psql -f $sqlscript

echo "Restarting postgres DB"
sudo /etc/init.d/postgresql restart
