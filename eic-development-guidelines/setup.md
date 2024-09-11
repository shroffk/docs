# Using Git on demo machines

### Intro 
ssh into your demo machine of choice.
``` bash
ssh jmaldonad@demo06.eic.bnl.gov
jmaldonad@demo06.eic.bnl.gov's password: 
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Fri Sep  6 11:31:44 2024 from 130.199.104.45
[jmaldonad@demo06 ~]$
```

If your github account does not have an ssh key associated with this machine and you try to git clone a project permission will be denied.

``` bash
[jmaldonad@demo06 project]$ git clone git@github.com:eicorg/ghaction.git
Cloning into 'ghaction'...

git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

```

