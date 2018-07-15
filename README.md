
Account Creation
===============

This role privisions new accounts in LDAP, sets their home directory quota on XFS NFS file server, sends user an initial password via challenge-response sms, and sends notification of completion via Slack.


Things To Consider
------------

Two types of accounts are created:

***normal***

***collabrator***

The only difference between the two would be size of XFS quota, 100G vs 2G, as well as email addresses stored in LDAP.  This resulted from an environment where non-standard users were granted access to enterprise resources.

In **scratch.yml** the role will also create defined directories on GPFS scratch.


Requirements
------------

You must have Ansible 2.0 installed.

You need an account with [Twilio](https://www.twilio.com/)

You need [Slack](https://slack.com/)

[OpenLDAP](https://www.openldap.org/) directory server

CentOS 6 or 7

XFS on your NFS servers

SSH keys and Sudo configured appropriately for all servers you're running the role upon

Manager access to your LDAP server or a service account with proper ACLs


TODO
--------------

In **create_user.yml** check for presence of ldif before removing

Populate gids from LDAP in **vars/main.yml** rather than defining them manually

Replace manual challenge response steps in **csms.yml** and **rsms.yml** with automated process such as application that verifies user identity

Break up variables in **vars/main.yml** into separate files as it's rather large.


Role Variables
--------------

Here are some variables in the role which can be passed at the command line using Ansible extra vars "-e var=blah:"
 
    host: blah                                 # Host to run play against which contains ldap tools - passed to create_user.yml
    user: blah                                 # User to create - passed to create_user.yml
    group: blah                                # Existing group in LDAP for user - passed to create_user.yml
    gecos: blah                                # First and Last name for user - passed to create_user.yml
    number: blah                               # Cell Phone to send SMS - passed to challenge.yml and rsms.yml


Note, .yml in roles/accounts/tasks will only run if conditionals [create, remove, email, csms, rsms] met so you must define them at command line:

    create: []                                 # Defining 'create' will run create_user.yml
    remove: []                                 # Defining 'remove' will run remove_user.yml
    email: []                                  # Defining 'email' will run email.yml
    csms: []                                   # Defining 'csms' will run challenge.yml
    rsms: []                                   # Defining 'rsms' will run rsms.yml


Examples
--------


Create user in LDAP:

```
ansible-playbook accounts.yml "-e hosts=gpfs_node -e user=user -e group=lab -e gecos='full name' -e create=[]" --sudo -K --ask-vault-pass
```

Remove user from LDAP:

```
ansible-playbook accounts.yml "-e hosts=gpfs_node -e user=user -e remove=[]" --sudo -K --ask-vault-pass
```


Set home directory for user:

```
ansible-playbook accounts.yml "-e hosts=nfs_server -e user=user-e group=lab -e create=[]" --sudo -K -t xfs --ask-vault-pass
ansible-playbook accounts.yml "-e hosts=nfs_server -e user=user -e group=lab -e collab=[]" --sudo -K -t xfs --ask-vault-pass
```

Send challenge:

```
ansible-playbook accounts.yml "-e hosts=gpfs_node -e csms=[] -e number=+1phone -e user=user" -t csms --sudo -K --ask-vault-pass
```

Send response:

```
ansible-playbook accounts.yml "-e hosts=gpfs_node -e rsms=[] -e number=+1phone -e user=user" -t rsms --sudo -K --ask-vault-pass
```



**Note, you can use a bash function in ~/.bashrc to make these easier to use.**




For example:

```
newuser() { ansible-playbook $ANSIBLE_HOME/accounts.yml "-e hosts=gpfs_node -e user=$1 -e group=$2 -e gecos='$3' -e create=[]" --sudo -K --ask-vault-pass ; }
rmuser() { ansible-playbook $ANSIBLE_HOME/accounts.yml "-e hosts=gpfs_node -e user=$1 -e remove=[]" --sudo -t remove -K --ask-vault-pass ; }
collab() { ansible-playbook $ANSIBLE_HOME/accounts.yml "-e hosts=gpfs_node -e user=$1 -e group=$2 -e gecos='$3' -e email=$4 -e collab=[]" --sudo -K --ask-vault-pass; }
csms() { ansible-playbook $ANSIBLE_HOME/accounts.yml "-e hosts=gpfs_node -e user=$1 -e number=+1$2 -e csms=[]" -t csms --sudo -K --ask-vault-pass ; }
rsms() { ansible-playbook $ANSIBLE_HOME/accounts.yml "-e hosts=gpfs_node -e user=$1 -e number=+1$2 -e rsms=[]" -t rsms --sudo -K --ask-vault-pass ; }
```

Thus to create a collab account type in bash:


```
collab user lab 'full name' email@example.com
```

License
-------

GPL

Author Information
------------------

Douglas Duckworth
quackmaster@protonmail.com
