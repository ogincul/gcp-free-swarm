# GCP free Docker swarm

1. Create two Ubuntu instances: halloumi-master, halloumi-slave

   - Region: us-central1 (Iowa), us-central1-a
   - Series: N1
   - Machine type: f1-micro
   - Boot disk: Ubuntu 16.04 LTS
   - Network: default, External IP: Ephemeral

2. Create in both accounts firewall rule: halloumi-firewall-ssh

   - Network: default
   - Targets: All instances in the network
   - Source IP ranges: 0.0.0.0/0
   - Specified protocols and ports: tcp 22

3. Install in both instances Docker

   ```
   - sudo apt-get update
   - sudo apt-get install docker.io
   ```

4. Create VPC networks: halloumi-vpc-master, halloumi-vpc-slave

   - Region: us-central1
   - IP address range: 10.2.0.0/28 (master), 10.2.0.16/28 (slave)

5. Create VPC network peering: halloumi-vpc-peer-master, halloumi-vpc-peer-slave

   - Your VPC network: halloumi-vpc-master/halloumi-vpc-slave
   - Peered VPC network: In another project
   - Project ID: balmy-amp-260313/orbital-age-220919
   - VPC network name: halloumi-vpc-slave/halloumi-vpc-master

6. Create firewall rules: halloumi-firewall-internal-all

   - Network: halloumi-vpc-master/halloumi-vpc-slave
   - Targets: All instances in the network
   - Source IP ranges: 0.0.0.0/0
   - Allow all

7. Stop both instances

8. Edit instances and update network interface for each:

   - Network: halloumi-vpc-master/halloumi-vpc-slave
   - Subnetwork: halloumi-vpc-master/halloumi-vpc-slave

9. Start both instances

10. Create the docker swarm

    ```
    # master, 10.2.0.3 is the master internal IP as seen in VM instances page
    sudo docker swarm init --advertise-addr 10.2.0.2
    
    # slave, join command as outputed by the earlier swarm init command
    sudo docker swarm join --token <token> 10.2.0.2:2377
    
    # nodes
    sudo docker node ls
    ```

11. Run and scale nginx from master node

    ```
    sudo docker service create -p 80:80 nginx
    sudo docker service ls
    sudo docker node ps halloumi-master
    sudo docker node ps halloumi-slave
    sudo docker service scale <SERVICE-ID>=2
    ```

12. Point domain to master External IP

13. Create firewall rules: halloumi-firewall-http

    - Network: halloumi-vpc-master/halloumi-vpc-slave
    - Targets: All instances in the network
    - Source IP ranges: 0.0.0.0/0
    - Specified protocols and ports: tcp 80
