# IP Request  
https://info.itd.bnl.gov/cgi-bin/ipdns/ipreg/single

# Create object in Centrify Access Manager -- needs domain admin privilages
Define hostname in zone:  
```
OU=EIC-CON,OU=EIC,OU=CCM,DC=bnl,DC=gov
```

# Then use the centrify tar from ITD to install client
```
wget https://mirror.bnl.gov/software/centrify/6.0.1/system-agent/centrify-rhel6-
x86_64.tar
tar  xvf centrify-rhel6-x86_64.tar
cd centrify-2023.1-rhel6-x86_64/
```
# install dependancies for RHEL9:
```
yum install perl-File-Copy  perl-Sys-Hostname perl-lib chkconfig
```

# run the install
./RUNME
#It will ask you for an admin user account and what zone to join to:
OU=EIC-CON,OU=EIC,OU=CCM,DC=bnl,DC=gov

# The script usually fails and the ITD solution is to force manually joining to the domain
```
/usr/sbin/adjoin -V -c "OU=EIC-CON,OU=EIC,OU=CCM,DC=bnl,DC=gov" --zone OU=EIC-CON,OU=EIC,OU=CCM,DC=bnl,DC=gov --user kkds  --force bnl.gov
```

## now you should have a working centrify login!!

## Setup Local sudoers

```
touch /etc/sudoers.d/domain_users

Enter this scope {
kkulmatycski ALL=(ALL) ALL
shroffk ALL=(ALL) ALL
seth ALL=(ALL) ALL
desilva ALL=(ALL) ALL
kabir ALL=(ALL) ALL
sclark ALL=(ALL) ALL
jmaldonad ALL=(ALL) ALL
olsen ALL=(ALL) ALL

#BNL\kkulmatycski ALL=(ALL) ALL
#kkulmatycski@bnl.gov ALL=(ALL) ALL
}
```

## Ansible Template:

```
---
- name: Configure sudo access for domain users
  hosts: all
  become: yes
  vars:
    sudo_users: |
      kkulmatycski ALL=(ALL) ALL
      shroffk ALL=(ALL) ALL
      seth ALL=(ALL) ALL
      desilva ALL=(ALL) ALL
      kabir ALL=(ALL) ALL
      sclark ALL=(ALL) ALL
      jmaldonad ALL=(ALL) ALL
      olsen ALL=(ALL) ALL

  tasks:
    - name: Ensure sudo is installed
      package:
        name: "{{ item }}"
        state: present
      with_items:
        - "{{ 'sudo' if ansible_os_family == 'Debian' else 'sudo' }}"
      when: ansible_os_family in ['Debian', 'RedHat']

    - name: Ensure /etc/sudoers.d directory exists
      file:
        path: /etc/sudoers.d
        state: directory
        mode: '0755'
        owner: root
        group: root

    - name: Create backup of existing file (if it exists)
      copy:
        src: /etc/sudoers.d/domain_users
        dest: /etc/sudoers.d/domain_users.bak
        remote_src: yes
      when: ansible_check_mode == false
      ignore_errors: yes

    - name: Create domain_users sudo file
      copy:
        content: "{{ sudo_users }}"
        dest: /etc/sudoers.d/domain_users
        mode: '0440'
        owner: root
        group: root
        validate: /usr/sbin/visudo -cf %s
      register: sudo_file

    - name: Verify sudo configuration
      command: sudo -v
      changed_when: false
      when: sudo_file.changed
```

