# aws_ec2_start
### Youtube Channel: https://www.youtube.com/c/TamilTutEra
### Video Link: https://youtu.be/7DPvDHsZQUw
## Tamiltutera Ansible in Tamil - How to start the AWS EC2 Instance using AWS CLI Ansible Playbook

Installing Pre-requisites for Ansible and Python in Linux RHEL
```
sudo yum update -y
sudo yum install python3-devel -y
sudo yum install ansible -y
sudo yum install boto3 -y
sudo yum install awscli -y
```

hosts
```
localhost
```

ansible.cfg
```
[defaults]
inventory = hosts
```

ansible-playbook main.yml -e "instance_id=<aws_instance_id> region=us-east-1"

```
---
- hosts: all
  gather_facts: false
  connection: local
  tasks:
    - name: Check current state of the EC2 Instance
      command: aws ec2 describe-instances --instance-ids {{ instance_id }} --region {{ region }} --output text --query 'Reservations[*].Instances[*].State.Name'
      register: instance_state
      delegate_to: localhost
    - debug:
        msg: "{{ instance_state.stdout }}"
      delegate_to: localhost 
    - fail:
        msg: "Instance is already running"
      when: instance_state.stdout == "running"
      delegate_to: localhost
    - set_fact:
        current_state: "{{ instance_state.stdout }}"
     # AWS Instance starting
    - name: starting the AWS EC2 Instances
      command: aws ec2 start-instances --instance-ids {{ instance_id }} --region {{ region }}
      when: instance_state.stdout != "running"
      register: start_instance_state
      delegate_to: localhost
    - name: wait for starting the instance
      wait_for:
        timeout: 60
    - name: checking the status of the instance
      debug:
        msg: "Instance started successfully"
      when: start_instance_state.changed == true
```
