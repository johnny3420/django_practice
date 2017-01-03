# django_practice
Practice building a django site

##Set environment

```
virtualenv -p `which python3` env
source env/bin/activate
```

##Downloand djangocms-installer and run

```sh
pip install djangocms-installer
djangocms -w cms_site
# Database configuration (in URL format). Example: sqlite://localhost/project.db [default sqlite://localhost/project.db]: 
# django CMS version (choices: 3.2, 3.3, 3.4, stable, develop) [default stable]: 
# Django version (choices: 1.8, 1.9, stable) [default stable]: 
# Activate Django I18N / L10N setting; this is automatically activated if more than language is provided (choices: yes, no) [default yes]: no
# Install and configure reversion support (only for django CMS 3.2 and 3.3) (choices: yes, no) [default yes]: 
# Languages to enable. Option can be provided multiple times, or as a comma separated list. Only language codes supported by Django can be used here. Example: en, fr-FR, it-IT: en
# Optional default time zone. Example: Europe/Rome [default America/Los_Angeles]: 
# Activate Django timezone support (choices: yes, no) [default yes]: 
# Activate CMS permission management (choices: yes, no) [default no]: 
# Use Twitter Bootstrap Theme (choices: yes, no) [default no]: no
# Use custom template set [default no]: 
# Load a starting page with examples after installation (english language only). Choose "no" if you use a custom template set. (choices: yes, no) [default no]:
# ...
# Creating admin user
# Username (leave blank to use 'john'): 
# Email address: jtdavis@ucdavis.edu
# Password: 
# Password (again): 
# Superuser created successfully.
# All done!
```

## Add password validation

```sh
pip install django-passwords
gedit env/lib/python3.5/site-packages/django/contrib/auth/forms.py
# Add from passwords.fields import PasswordField to the beginning of the file
# Under class UserCreationForm
# Change password1 = forms.CharField(label=_("Password"), and password2 = forms.CharField(label=_("Password confirmation"),
# to
# password1 = PasswordField(label=_("Password"), and password2 = PasswordField(label=_("Password confirmation"),

# Under class SetPasswordForm
# Change new_password1 = forms.CharField(label=_("New password"), and new_password2 = forms.CharField(label=_("New password confirmation"),
# to
# new_password1 = PasswordField(label=_("New password"), and new_password2 = PasswordField(label=_("new Password confirmation"),


# add this to file
PASSWORD_COMPLEXITY = { # You can omit any or all of these for no limit for that particular set
    "UPPER": 1,        # Uppercase
    "DIGITS": 1,       # Digits)
}

# Change PASSWORD_COMPLEXITY = getattr(
#    settings, "PASSWORD_COMPLEXITY", None )
# to
# PASSWORD_COMPLEXITY = getattr(
#    settings, "PASSWORD_COMPLEXITY", PASSWORD_COMPLEXITY)
```
