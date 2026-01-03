How to automate RabbitMQ cluster creation with Docker using Ansible on multiple instances at once.
==================================================================================================


*   We’ll keep once instances as **control node** and other as **managed nodes**.
*   Let’s create three instances.

![Creating instances](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*OauaE_wo8HEY5TQePPenFw.png)

*   Name your instances as **ansible-demo-control** (for control node) and **ansible-demo-managed1** and **ansible-demo-managed2**. (from other two managed nodes)

![Naming instances.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Rfbe8XZPE9vR1AcBy7xu2g.png)

### 2. SSH into managed node from control node.

*   There are two ways to **ssh** into managed nodes:
*   **First:** As we are using the same key-pair for all three instances (example-keypair) we have to just copy the keypair to the control node using **scp.**
*   **Second:** Alternate way if you are not using AWS or keypair, you have to create one using **ssh-keygen** command. Copy the **public key** to the managed nodes and keep the **private key** on the control node to ssh into managed nodes.
*   Let’s copy the keypair to the control node.

![Copying keypair to control node](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*IgjTnyhD4hqEUFzlem1OdA.png)

*   As you can see we have copied our keypair to the control node.

![Keypair present on control node.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*67AhHQUpFUlcGTyw661DcQ.png)

*   Let’s give necessary permission to the keypair.

![Giving permission to keypair](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*wswbXGk6EvSq0gC2yM-Z0w.png)

### 3. Installing ansible on control node.

*   We just need to install **ansible** on the **control node**
*   No need to install it on the **managed nodes**
*   Let’s install ansible using below commands

```
sudo apt update -y &&
sudo apt install software-properties-common -y &&
sudo add-apt-repository --yes --update ppa:ansible/ansible -y &&
sudo apt install ansible -y
```

*   Check ansible version to ensure installation

![Ansible version check](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*V55ZjPDW1dTghXoFdUVNTA.png)

### 4. Create ansible inventory file

*   To make connection to the managed nodes, we have to create a inventory file in which we will the information for connection as well as the hostname.
*   Note: use private IPs of the managed nodes as public IPs keeps changing

![inventory file for ansible](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*eeHkzloxP6gwQDfjx02BKw.png)

*   Now put the ansible private key file (example-keypair) in the **ansible.cfg** so that it will fetch the private key from there.
*   Edit the **ansible.cfg** and append this three lines.

```
vi /etc/ansible/ansible.cfg

[defaults]
PRIVATE_KEY_FILE=/home/ubuntu/example-keypair.pem
ANSIBLE_PYTHON_INTERPRETER=/usr/bin/python3
INVENTORY=/home/ubuntu/inventory
``````

**Note:** The variables should be same as mentioned above.

*   We are done with the setup!

### 5. Test connection with ansible ad-hoc command

*   Test with a random shell command
*   **Note:** It will ask you to permanently store in the host list, confirm with typing “**yes**” for each managed node.

![checking connection with managed nodes.](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*QxMoj2zzjxOaGOjVy935Gw.png)

*   Boohoo!! we are done with setup and connection with managed nodes.

### 6. Now let’s create necessary files for RabbitMQ cluster creation.

*   Refer to this GitHub repository for creation of files including **ansible playbook**, **docker-compose.yml.j2**, **ansible_script.sh** and the **inventory file**: [https://github.com/ulhas001/RabbitMQ_Cluster_with_Ansible](https://github.com/ulhas001/RabbitMQ_Cluster_with_Ansible/tree/main)
*   Understand the structure of the files carefully as for your case multiple nodes could be there so edit the files accordingly.

### 7. Explanation of files

*   **ansible_script.sh**: It is a script written to form a cluster between managed nodes i.e. master and slaves nodes. Carefully edit the file as per your instance’s IP addresses and directories. You can add as many nodes as slaves to connect to the master in the last piece of code where the **rabbit2** activities are mentioned.
*   **docker-compose.j2.yml**: This is a jinja template to dynamically pass the hostnames of our instances, so that we can join cluster for many slaves nodes using thier hostnames. Focus on the volumes mount for erlang cookie and data. This is necessary to run RabbitMQ.
*   **inventory:** you can name it anything, but important things is the IP address and hostname of your instances, because this is the file which control node will take reference of, to execute the commands and automation.
*   **rabbitmq_playbook.yml:** This is one of the important files as it is installing docker, docker-compose, and other plugins so that you don’t have to on the managed nodes, also it is copying the docker-compose.yml file with hostname of particular managed node on which is it executing. Also pulling the image and running the container has being taken care of.

> **NOTE: Please open necessary ports in the same security group required for rabbitMQ to function, otherwise you’ll face unnecessary errors. Refer the article for information about clustering port to enable:** [https://www.rabbitmq.com/docs/networking#ports](https://www.rabbitmq.com/docs/networking#ports)

### 8. Run the ansible_script.sh

*   There you go !!!
*   Your cluster is created

![Ensuring RabbitMQ cluster creation](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*7HATnGOthHQDaWfQO32evQ.png)

Thank you!!

Keep Learning And Keep Sharing!👍😁
