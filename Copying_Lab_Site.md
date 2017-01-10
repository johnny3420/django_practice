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
# Add from passwords.fields import PasswordField to the beginning of the file
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



#### Build the database and start the server ( Will not run on django >= 1.8)

```sh
python manage.py makemigrations lab_members cms_lab_members cms_lab_carousel cms_lab_publications cms_shiny cms_lab_data cms_genome_browser
python manage.py makemigrations
python manage.py migrate

python manage.py createsuperuser
python manage.py runserver 0:8000
```


#### Add content

- Go to `http://symposium.plb.ucdavis.edu/`, login, and start adding pages. Some of the CMS lab components will be accessible via the CMS Plugin menu and others will be used as CMS Apps.

  - To add a CMS Plugin: edit a page in `Structure` mode, hover over the icon for adding a plugin to a content block, scroll down to `Lab Plugins`, and select the desired plugin.

  - To add a CMS App: create a new page, go to advanced page settings, and select the app from the dropdownwn menu labeled `Application:`. Where available, you can also attach a relevant menu for editing page contents from the dropdownwn menu labeled `Attached menu:`.
