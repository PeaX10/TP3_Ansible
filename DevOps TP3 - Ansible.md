# DevOps: TP3 - Ansible

## Inventories  
Default folder: ``/etc/ansible/hosts``  
```yaml
all :
    vars :
        ansible_user : centos
        ansible_ssh_private_key_file : /path/to/private/key
    children :
        prod :
            hosts : OVH server IP
```  
Then once we ping we get the pong:  
![ping return](https://raw.githubusercontent.com/PeaX10/TP3_Ansible/master/img/ping_return.png)  
  
  
Now let's remove our Http server:  
![removing http](https://raw.githubusercontent.com/PeaX10/TP3_Ansible/master/img/remove_httpd.png)  
  

## Playbooks
Now here is our first playbook, yay !  
```yaml
- hosts : all
gather_facts : false
become : yes

tasks :
    - name : Test connection
      ping :
```  
Let's check the syntaxe then execute it:  
![check syntaxe playbook](https://raw.githubusercontent.com/PeaX10/TP3_Ansible/master/img/syntaxe_check.png)
![first playbook](https://raw.githubusercontent.com/PeaX10/TP3_Ansible/master/img/first_playbook.png)  
  
## Advanced Playbooks
```yaml
- hosts : all
gather_facts : false
become : yes
tasks :
    # Install Docker
    - name : Install yum-utils
    yum :
        name : yum-utils
        state : latest
    - name : Install device-mapper-persistent-data
    yum :
        name : device-mapper-persistent-data
        state : latest
    - name : Install lvm2
    yum :
        name : lvm2
        state : latest
    - name : Add Docker stable repository
    yum_repository :
        name : docker-ce
        description : Docker CE Stable - $basearch
        baseurl : https://download.docker.com/linux/centos/7/$basearch/stable
        state : present
        enabled : yes
        gpgcheck : yes
        gpgkey : https://download.docker.com/linux/centos/gpg
    - name : Install Docker
    yum :
        name : docker-ce
        state : present
    - name : Make sure Docker is running
        service : name=docker state=started
        tags : docker
```  
Now let's execute it: 
![advancedplaybook](https://raw.githubusercontent.com/PeaX10/TP3_Ansible/master/img/advanced_playbook.png)

*__What is $basesearch ?__*  
When a repository id is displayed, append these yum variables to the string if they are used in the baseurl/etc. Variables are appended in the order listed (and found).  
  
## Roles
```yaml
- name: Close All ports
  iptables:
    chain: INPUT
    policy: DROP
- name: Open HTTP port (80)
  firewalld:
    service: http
    permanent: yes
    state: enabled
- name: Open HTTP port (22)
  firewalld:
    service: ssh
    permanent: yes
    state: enabled
```  
For HTTPS traffic, you should open 443 TCP port.  
  
## Deploy our app
Ansible .yml file.
```yaml
---
- name: Install yum utils
  yum:
    name: yum-utils
    state: latest

- name: Install device-mapper-persistent-data
  yum:
    name: device-mapper-persistent-data
    state: latest- name: Install lvm2
  yum:
    name: lvm2
    state: latest

- name: Add Docker repo
  get_url:
    url: https://download.docker.com/linux/centos/docker-ce.repo
    dest: /etc/yum.repos.d/docker-ce.repo
  become: yes

- name: Install Docker
  package:
    name: docker-ce
    state: latest
  become: yes

- name: Start Docker service
  service:
    name: docker
    state: started
    enabled: yes
  become: yes

- name: Add user centos to docker group
  user:
    name: centos
    groups: docker
    append: yes
  become: yes

- name: Create a network
  docker_network:
    name: my_net

- name: Run POSTGRES
  docker_container:
    name: postgres
    image: postgres:12.0
    env:
      POSTGRES_DB: db_test
      POSTGRES_USER: guest
      POSTGRES_PASSWORD: guest_pwd
    networks:
      - name: "my_net"

- name: Run API
  docker_container:
    name: http_api
    image: peax10/tp-cpe:http
    env:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/db_test
      SPRING_DATASOURCE_USERNAME: guest
      SPRING_DATASOURCE_PASSWORD: guest_pwd
    networks:
      - name: "my_net"

- name: Run DB Changelog
  docker_container:
    name: db
    image: peax10/tp-cpe:db
    env:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/db_test
      SPRING_DATASOURCE_USERNAME: guest
      SPRING_DATASOURCE_PASSWORD: guest_pwd
    networks:
      - name: "my_net"

- name: Run HTTPD
  docker_container:
    name: httpd
    image: peax10/tp-cpe:httpd
    ports:
      - "80:80"
    networks:
      - name: "my_net"
```  
The only port we need to open here is the 80. The 8080 and 5432 are already accessible between containers because they are all in the same network.  
``SPRING_DATASOURCE_URL`` refer to the application.yml file. It will override the value located in:  
```yaml
spring:
    datasource:
        url: URL replace
```
## Continuous Deployment
