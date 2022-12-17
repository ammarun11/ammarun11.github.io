---
title: "[Odoo] Setup odoo 12 On Ubuntu 16.04 LTS"
date: 2022-12-17
categories: [ngoprek, server, cloud, odoo, ubuntu]
tags:
  - Jekyll
  - update
---

### Step 1# Update apt source list and install updates

```
sudo apt-get update
sudo apt-get upgrade
```

### Step 2# Install PostgreSQL 9.6+ and create database user

```
sudo apt-get install python-software-properties
sudo vim /etc/apt/sources.list.d/pgdg.list
```

Add a line for the repository

```
deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main
```

Import the repository signing key, and update the package lists

```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
```

Ubuntu includes PostgreSQL by default. To install PostgreSQL on Ubuntu, use the _apt-get_ (or other apt-driving) command:

```
sudo apt-get install postgresql-9.6
```

Create Database user for Odoo

```
sudo su - postgres -c "createuser -s odoo" 2> /dev/null || true
```

### Step 3# Install tool packages

```
sudo apt-get install wget git python3-pip gdebi-core -y
```

### Step 4# Install python packages for Odoo 12

```
pip3 install Babel decorator docutils ebaysdk feedparser gevent greenlet html2text Jinja2 lxml Mako MarkupSafe mock num2words ofxparse passlib Pillow psutil psycogreen psycopg2 pydot pyparsing PyPDF2 pyserial python-dateutil python-openid pytz pyusb PyYAML qrcode reportlab requests six suds-jurko vatnumber vobject Werkzeug XlsxWriter xlwt xlrd gdata
```

### Step 5# Install odoo 12 

```
wget -c https://nightly.odoo.com/12.0/nightly/deb/odoo_12.0.latest_all.deb

dpkg -i odoo_12.0.latest_all.deb
```

Go to web browser to access Odoo

```
http://localhost:8069
```

Your Feedback will be Appreciated. Thanks
# Happy,  Enjoy Ngoprek ~
