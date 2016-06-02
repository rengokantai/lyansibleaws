#### lyansibleaws
#####Installation and Configuration
######Creating the Environment - Control Server and Nodes
create a control machine, using password only donot use ssh.
```
adduser test
ssh-keygen (here need to copy local key and pri key)
ssh-copy-id test@vultr.guest
```
then config visudo, config ansible.cfg,config hosts
```
[control]
vultr.guest
```
######Ansible and AWS Configuration - Environment Variables
check first:
```
env | grep AWS
aws ec2 describe-regions
export AWS_ACCESS_KEY_ID='A'
export AWS_SECERT_ACCESS_KEY='a'
```
#####playbook
######Ansible Playbook - Instance Run (we use aws ami (redhat))
playbook
```
---
- hosts: aws
  remote_user: ec2-user
  become: yes
  connection: ssh
  gather_facts: yes
  tasks:
  - name: Exe
    shell: ls -la
    register: result
  - name: display
    debug: var=result
```
hosts
```
[control]
vultr.guest
[aws]
ec2-5.compute-1.amazonaws.com
```
we need to use pem file,so must use ssh-agent
```
ssh-agent bash
cp ~/mykey.pem . //need to change the permission of the file.
ssh-add mykey.pem
ansible-playbook shell.yml
exit
```
###### Module: EC2_Facts
cookbook:
```
---
- hosts: aws
  remote_user: ec2-user
  become: yes
  connection: ssh
  gather_facts: yes
  tasks:
  - name: ec2facts
    action: ec2_facts
  - name: display instype if is t2.micro
    debug: msg='{{ansible_ec2_instance_type}}'
    when: ansible_ec2_instance_type != 't2.micro'
```
######Module: EC2_Key
first
```
sudo pip install boto
```
playbook
```
---
- hosts: localhost
  connection: local
  remote_user: test
  become: yes
  gather_facts: no
  vars_files:
  - cred.yml
  tasks:
  - name: create new key
    ec2_key:
      aws_access_key: '{{aws_id}}'
      aws_secret_key: '{{aws_key}}'
      name: awskey
      region: us-east-1
      state: present/absent
```
cred.yml
```
aws_id: qwe
aws_key: 123
region: us-east-1
subnetid:
```
###### Module: EC2 - Managing Instance State
start,stop,terminate
```
---
- hosts: localhost
  connection: local
  remote_user: test
  become: yes
  gather_facts: no
  vars_files:
  - cred.yml
  tasks:
  - name: change state
    ec2:
      aws_access_key: '{{aws_id}}'
      aws_secret_key: '{{aws_key}}'
      region: '{{region}}'
      image: ami-f
      instance_ids: i-1
      state: present

```
######Module: EC2 - Provisioning New Instances
```
---
- hosts: localhost
  connection: local
  remote_user: test
  become: yes
  gather_facts: no
  vars_files:
  - cred.yml
  tasks:
  - name: basic
    ec2:
      aws_access_key: '{{aws_id}}'
      aws_secret_key: '{{aws_key}}'
      region: '{{region}}'
      image: ami-f5f41398
      vpc_subnet_id: '{{subnetid}}'
      instance_type: t2.micro
      count: 2
      assign_public_ip: yes
```
######Module: EC2_AMI - Basic Creation
```
---
- hosts: localhost
  connection: local
  remote_user: test
  become: yes
  gather_facts: no
  vars_files:
  - cred.yml
  tasks:
  - name: basic creation
    ec2_ami:
      aws_access_key: '{{aws_id}}'
      aws_secret_key: '{{aws_key}}'
      region: '{{region}}'
      instance_id: i-02855a9e
      wait: no
      name: myami
      tags:
        Name: Myami
        Service: Test
    register: instance
```
###### Module: EC2_AMI - Customization
```
- hosts: localhost
  connection: local
  remote_user: test
  become: yes
  gather_facts: no
  vars_files:
  - cred.yml
  tasks:
  - name: basic creation
    ec2_ami:
      aws_access_key: '{{aws_id}}'
      aws_secret_key: '{{aws_key}}'
      region: '{{region}}'
      instance_id: i-02855a9e
      wait: no
      name: myami2
      tags:
        Name: Myami2
        Service: Test2
      device_mapping:
        - device_name: /dev/sdb1
          size: 100
          delete_on_termination: false
          volume_type: gp2
    register: instance
```
