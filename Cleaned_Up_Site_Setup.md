#### Need python 3.4, 3.3 and 3.5 will not work

```sh
sudo -s
cd /usr/lib
mkdir python3.4.2
cd /usr/src
wget https://www.python.org/ftp/python/3.4.2/Python-3.4.2.tgz
tar -zxvf Python-3.4.2.tgz 
cd Python-3.4.2/
./configure --prefix=/usr/lib/python3.4.2
make
make install
```

#### Clone the `djangocms-lab-site` repository

```sh
PROJECT_NAME=rhizobiomics_site
PROJECT_DIR=/mnt/data/www/$PROJECT_NAME
mkdir $PROJECT_DIR
cd $PROJECT_DIR
git clone https://github.com/mfcovington/djangocms-lab-site.git .
```

#### Customize the lab name in `cms_lab_site/settings/base.py`

    LAB_NAME = 'Rhizobiomics'
    
#### Change `cms_lab_site/settings/__init__.py` from:

`from .development import *`
to:
`from .production import *`


#### Make a virtual environment for the project and install all of the dependencies

```sh
virtualenv -p /usr/lib/python3.4.2/bin/python3 env

source env/bin/activate
pip install --upgrade pip
pip install -r requirements-for-quick-install.txt
pip install djangocms-lab-site.components-for-installation/*

# install bootstrap, jquery, etc. using bower (to install bower: https://bower.io/#install-bower)
# cannot be done in sudo
exit
cd /mnt/data/www/
sudo chown -R jtdavis rhizobiomics_site
cd rhizobiomics_site
bower install
cd ..
sudo chown -R root rhizobiomics_site
```

#### Add password validation

```sh
sudo -s
cd /mnt/data/www/rhizobiomics_site
source env/bin/activate
pip install django-passwords
nano env/lib/python3.4/site-packages/django/contrib/auth/forms.py
# Add `from passwords.fields import PasswordField` to the beginning of the file
# Under class UserCreationForm
# Change password1 = forms.CharField(label=_("Password"), and password2 = forms.CharField(label=_("Password confirmation"),
# to
# password1 = PasswordField(label=_("Password"), and password2 = PasswordField(label=_("Password confirmation"),

# Under class SetPasswordForm
# Change new_password1 = forms.CharField(label=_("New password"), and new_password2 = forms.CharField(label=_("New password confirmation"),
# to
# new_password1 = PasswordField(label=_("New password"), and new_password2 = PasswordField(label=_("new Password confirmation"),

# Under class AdminPasswordChangeForm
# Change password1 = forms.CharField(label=_("Password"), and password2 = forms.CharField(label=_("Password (again)"),
# to
# password1 = PasswordField(label=_("Password"), and password2 = PasswordField(label=_("Password confirmation"),

nano env/lib/python3.4/site-packages/passwords/validators.py
# add this to file
PASSWORD_COMPLEXITY = { # You can omit any or all of these for no limit for that particular set
    "UPPER": 1,        # Uppercase
    "DIGITS": 1,       # Digits)
}

#PASSWORD_MIN_LENGTH = getattr(
#    settings, "PASSWORD_MIN_LENGTH", 6)
#to
#PASSWORD_MIN_LENGTH = getattr(
#    settings, "PASSWORD_MIN_LENGTH", 8)

# Change PASSWORD_COMPLEXITY = getattr(
#    settings, "PASSWORD_COMPLEXITY", None )
# to
# PASSWORD_COMPLEXITY = getattr(
#    settings, "PASSWORD_COMPLEXITY", PASSWORD_COMPLEXITY)
```

#### Version checking

When I building this site, I checked the versions of django CMS, Django, and Python:

```sh
python -c '
from cms import __version__
import django
import sys

print("django CMS: %s" % __version__)
print("Django:     %s" % django.get_version())
print("Python:     %s" % sys.version)
'
```

Results:

    django CMS: 3.1.0
    Django:     1.7.8
    Python:     3.4.2 (default, Jan  9 2017, 14:57:12) 
    [GCC 4.7.2]
    
 
 ## Create and configure PostgreSQL database

Create database and admin

```sh
sudo su - postgres
createdb rhizobiomics_db
createuser -P
# Enter name of role to add: rhizobiomics_admin
# Enter password for new role: 
# Enter it again: 
# Shall the new role be a superuser? (y/n) n
# Shall the new role be allowed to create databases? (y/n) n
# Shall the new role be allowed to create more new roles? (y/n) n
psql
```

Grant privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE rhizobiomics_db TO rhizobiomics_admin;
\q
```

Logout postgres user

```sh
exit
```

## Configure project to use PostgreSQL database

Set the project's database to the PostgreSQL database `rhizobiomics_db` in `/mnt/data/www/rhizobiomics_site/cms_lab_site/settings/production.py` by changing this:


```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'maloof_lab_site_db',
        'USER': 'maloof_lab_site_admin',
        # password will passed from environmental variable:
        'PASSWORD': os.environ.get("MALOOF_LAB_SITE_DB_PASSWORD", ''),
        'HOST': '127.0.0.1',
        'PORT': '5432', 
    }   
}
```

To


```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'rhizobiomics_db',
        'USER': 'rhizobiomics_admin',
        # password will passed from environmental variable:
        'PASSWORD': os.environ.get('RHIZOBIOMICS_DB_PASSWORD', ''),
        'HOST': '127.0.0.1',
        'PORT': '5432', 
    }   
}
```

## Configure site in Apache and set DB password

Create and add the following eight lines to `/etc/apache2/sites-available/rhizobiomics.org.conf` (with the actual password and secret key instead of 'super_secret_password' and 'super_secret_key'):

```apache

<VirtualHost *:80>
    ServerName rhizobiomics.org
    RedirectPermanent / http://phytonetworks.org/
</VirtualHost>


<VirtualHost *:80>
...
    WSGIDaemonProcess rhizobiomics_site python-path=/mnt/data/www/rhizobiomics_site:/mnt/data/www/rhizobiomics/env/lib/python3.4/site-packages
    <Location /rhizobiomics_site>
        WSGIProcessGroup rhizobioics_site
    </Location>
    SetEnv RHIZOBIOMICS_SECRET_KEY super_secret_key
    SetEnv RHIZOBIOMICS_DB_PASSWORD super_secret_password
    WSGIScriptAlias / /mnt/data/www/rhizobiomics_site/cms_lab_site/wsgi.py
    Alias /static/Rhizobiomics /mnt/data/www/rhizobiomics_site/static/
    Alias /media/Rhizobiomics /mnt/data/www/rhizobiomics_site/media/

...
</VirtualHost>
```

The new secret key was made using an [online generator](http://www.miniwebtool.com/django-secret-key-generator/).

Change `$PROJECT_DIR/cms_lab_site/wsgi.py` (`/mnt/data/www/rhizobiomics_site/cms_lab_site/wsgi.py`)to:

```python
"""
WSGI config for cms_lab_site project.

It exposes the WSGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/1.7/howto/deployment/wsgi/
"""

from django.core.handlers.wsgi import WSGIHandler
import django
import os

class WSGIEnvironment(WSGIHandler):

    def __call__(self, environ, start_response):

        os.environ['RHIZOBIOMICS_SECRET_KEY'] = environ['RHIZOBIOMICS_SECRET_KEY']
        os.environ['RHIZOBIOMICS_DB_PASSWORD'] = environ['RHIZOBIOMICS_DB_PASSWORD']
        os.environ.setdefault("DJANGO_SETTINGS_MODULE", "cms_lab_site.settings")
        django.setup()
        return super(WSGIEnvironment, self).__call__(environ, start_response)

application = WSGIEnvironment()
```

## Hide secret key

Change the `SECRET_KEY` variable in `/mnt/data/www/rhizobiomics_site/cms_lab_site/settings/production.py` from this:

```python
SECRET_KEY = os.environ.get('MALOOF_LAB_SECRET_KEY", '')
```

to this:

```python
SECRET_KEY = os.environ.get('RHIZOBIOMICS_SECRET_KEY", '')
```


## Create a secret file to hold secrets

In order to do migrations (and some other things), the secret key and the database password need to be set as environmental variables. However, the way we are passing them from Apache's configuration to `settings.py` via `wsgi.py` prevents them from being set for everyday use.

We can get around this by placing an `export` statement for each secret variable into a file (`secrets.txt` located in the production directory) that is only readable by root. This is file will be gitignored.

```sh
sudo su - root
SECRETS_FILE=/mnt/data/www/rhizobiomics_site/secrets.txt

echo 'export RHIZOBIOMICS_SECRET_KEY="<super_secret_key>"
export RHIZOBIOMICS_DB_PASSWORD="<super_secret_password>"' > $SECRETS_FILE

chmod 600 $SECRETS_FILE
exit
```

## Collect static files

Gitkeep empty static directory

```sh
mkdir static
touch static/.gitkeep
```
Collect the static files. This can be done whenever there are changes/additions to the static files.

```sh
sudo -s
cd /mnt/data/www/rhizobiomics_site
source env/bin/activate
source secrets.txt
./manage.py collectstatic
exit
```

## Migrate database and create super user

To perform the initial database migration, we do the following:

```sh
cd /mnt/data/www/rhizobiomics_site
source env/bin/activate
eval `sudo cat secrets.txt`
./manage.py makemigrations lab_members cms_lab_members cms_lab_carousel cms_lab_publications cms_shiny cms_lab_data cms_genome_browser
./manage.py makemigrations
./manage.py migrate
```

- If using Apache, be sure to run `/usr/sbin/apachectl -k restart` after making any such changes.

And we need to create a super user:

```sh
./manage.py createsuperuser
```

>     Username (leave blank to use 'jtdavis'): 
>     Email address: jtdavis@ucdavis.edu
>     Password: 
>     Password (again): 
>     Superuser created successfully.

Change the`ALLOWED_HOSTS` variables in `/mnt/data/www/rhizobiomics_site/cms_lab_site/settings/production.py` to this:

```python
DEBUG = False
...
ALLOWED_HOSTS = ['.plb.ucdavis.edu',
                 'rhizobiomics.org']
```

#### Add a wiki to site

Add the following to `/mnt/data/www/rhizobiomics/cms_lab_site/urls.py`

At the top

`from wiki.urls import get_pattern as get_wiki_pattern`
`from django_nyt.urls import get_pattern as get_nyt_pattern`

Under urlpatterns
    ...
    url(r'^notifications/', get_nyt_pattern()),
    url(r'^wiki/', get_wiki_pattern()),
    ...


Add the following to `/mnt/data/www/rhizobiomics/cms_lab_site/settings/base.py`

Inside THIRD_PARTY_APPS

    'django_nyt',
    'sorl.thumbnail',
    'wiki',
    'wiki.plugins.attachments',
    'wiki.plugins.notifications',
    'wiki.plugins.images',
    'wiki.plugins.macros',

Then add it to database

```sh
python manage.py syncdb
python manage.py migrate
```

#### Add content

- Go to `http://127.0.0.1:8000/`, login, and start adding pages. Some of the CMS lab components will be accessible via the CMS Plugin menu and others will be used as CMS Apps.

  - To add a CMS Plugin: edit a page in `Structure` mode, hover over the icon for adding a plugin to a content block, scroll down to `Lab Plugins`, and select the desired plugin.

  - To add a CMS App: create a new page, go to advanced page settings, and select the app from the dropdownwn menu labeled `Application:`. Where available, you can also attach a relevant menu for editing page contents from the dropdownwn menu labeled `Attached menu:`.
