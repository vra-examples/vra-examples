---
title: "Ubuntu Docker Host"
date: 2020/11/04 14:21
anchor: "ubuntudockerhost"
---
Deploy an Ubuntu 18.04 template and install Docker. Configure the Docker service and firewall to allow remote connections.

```yaml
formatVersion: 1
inputs:
  user:
    type: string
    title: Username for SSH
    description: The username you would like to have for the installation.
    default: demouser
  password:
    type: string
    pattern: '[a-z0-9A-Z@#$]+'
    encrypted: true
    title: Admin Account Password
    description: The password you would like to use for the System.
resources:
  dockerhost:
    type: Cloud.Machine
    properties:
      image: 'ubuntu1804'
      flavor: medium
      constraints:
        - tag: 'cloud:vsphere'
      cloudConfig: |
        #cloud-config
        users:
          - name: ${input.user}
            sudo: ['ALL=(ALL) NOPASSWD:ALL']
            groups: sudo
            shell: /bin/bash
        hostname: '${input.user}-dockerhost'
        packages:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg-agent
          - software-properties-common
        runcmd:
          - PASS=${input.password}    
          - USER=${input.user}
          - echo $USER:$PASS | /usr/sbin/chpasswd
          - sed -i "s/PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config
          - service ssh reload
          - curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
          - add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
          - apt-get update
          - apt-get install -yq docker-ce docker-ce-cli containerd.io
          - mkdir -p /etc/systemd/system/docker.service.d/
          - echo "[Service]" > /etc/systemd/system/docker.service.d/override.conf
          - echo "ExecStart=" >> /etc/systemd/system/docker.service.d/override.conf
          - echo "ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375" >> /etc/systemd/system/docker.service.d/override.conf
          - systemctl daemon-reload
          - systemctl restart docker.service
          - ufw allow 2375
          - ufw reload
      cloudConfigSettings: 
        phoneHomeTimeoutSeconds: 300
        phoneHomeShouldWait: true
        phoneHomeFailOnTimeout: true
```