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
virtualenv -p /home/john/Documents/Lab_Site/.localpython/bin/python3 env
source env/bin/activate

pip install --upgrade pip
pip install -r requirements-for-quick-install.txt
pip install djangocms-lab-site.components-for-installation/*

# install bootstrap, jquery, etc. using bower (to install bower: https://bower.io/#install-bower)
# cannot be done in sudo
cd ..
sudo chown -R jtdavis rhizobiomics_site
cd rhizobiomics_site
bower install
cd ..
sudo chown -R root rhizobiomics_site
```


#### Build the database and start the server

```sh
python manage.py makemigrations lab_members cms_lab_members cms_lab_carousel cms_lab_publications cms_shiny cms_lab_data cms_genome_browser
python manage.py makemigrations
python manage.py migrate

python manage.py createsuperuser
python manage.py runserver
```
