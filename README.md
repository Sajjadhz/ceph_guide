Below is a step-by-step guide to install Ceph on three Rocky Linux nodes using Cephadm. 

### Prerequisites
1. Three Rocky Linux nodes.
2. Root or sudo access to all nodes.
3. Networking setup to allow all nodes to communicate with each other.
4. Time synchronization (e.g., via NTP).
5. SSH access configured between the nodes.

### Step 1: Prepare Nodes

#### On all nodes:
1. Update all packages and reboot if necessary:
    ```bash
    sudo dnf update -y
    ```

2. Install necessary packages:
    ```bash
    sudo dnf install -y chrony lvm2 podman python3.9
    sudo systemctl enable --now chronyd
    ```

3. Set up passwordless SSH access:
    ```bash
    ssh-keygen -t rsa
    ssh-copy-id <user>@<node1>
    ssh-copy-id <user>@<node2>
    ssh-copy-id <user>@<node3>
    ```

4. Stop and disable firewall
    ```bash
    systemctl stop firewalld.service
    systemctl disable firewalld.service
    ```

5. Configure time synchronization
    ```bash
    cat >> /etc/chrony.conf <<EOF
    allow 192.168.0.0/16
    server 192.168.56.7 iburst  # replace this ip address with your node ip address
    EOF
    ```
    ```bash
    systemctl restart chronyd
    systemctl status chronyd
    chronyc sources
    timedatectl status
    timedatectl set-timezone Asia/Tehran
    timedatectl status
    date
    ```

### Step 2: Install and Configure Cephadm on the First Node

#### On the first node:
1. Download and make Cephadm executable:
    ```bash
    CEPH_RELEASE=18.2.4 # replace this with the active release
    curl --silent --remote-name --location https://download.ceph.com/rpm-${CEPH_RELEASE}/el9/noarch/cephadm
    chmod +x cephadm
    ```

2. Add Ceph repository:
    ```bash
    sudo ./cephadm add-repo --release reef # replace this with the active release
    sudo ./cephadm install
    ```

3. Bootstrap the cluster:
    ```bash
    sudo ./cephadm bootstrap --mon-ip <IP_OF_FIRST_NODE>
    ```

4. Copy the SSH key to other nodes:
    ```bash
    sudo cephadm shell
    ceph cephadm get-pub-key > ~/ceph.pub
    ssh-copy-id -f -i ~/ceph.pub root@<node2>
    ssh-copy-id -f -i ~/ceph.pub root@<node3>
    ```

### Step 3: Add Other Nodes to the Cluster

1. Add the nodes:
    ```bash
    sudo ceph orch host add <node2> <IP_OF_NODE2>
    sudo ceph orch host add <node3> <IP_OF_NODE3>
    ```

2. Verify hosts are added:
    ```bash
    sudo ceph orch host ls
    ```

### Step 4: Deploy Monitor and Manager Daemons

1. Deploy additional monitors (if needed):
    ```bash
    sudo ceph orch apply mon --placement="node2,node3"
    ```

2. Deploy manager daemons:
    ```bash
    sudo ceph orch apply mgr --placement="node1,node2,node3"
    ```

### Step 5: Prepare and Activate OSDs

#### On each node:
1. Identify disks to use for OSDs:
    ```bash
    lsblk
    ```

2. Create OSDs (replace `/dev/sdX` with actual disk):
    ```bash
    sudo ceph orch daemon add osd <node1>:/dev/sdX
    sudo ceph orch daemon add osd <node2>:/dev/sdY
    sudo ceph orch daemon add osd <node3>:/dev/sdZ
    ```

### Step 6: Verify Cluster Health

1. Check the status of the cluster:
    ```bash
    sudo ceph -s
    ```

### Step 7: Deploy Other Ceph Services

1. Deploy Metadata Server (MDS) for CephFS:
    ```bash
    sudo ceph orch apply mds fs_name --placement="node1,node2"
    ```

2. Deploy RGW (RADOS Gateway) for object storage:
    ```bash
    sudo ceph orch apply rgw rgw_name --placement="node1,node2,node3"
    ```

### Step 8: Access the Ceph Dashboard

1. Retrieve the URL and admin credentials for the Ceph Dashboard:
    ```bash
    ceph mgr services
    ceph dashboard create-self-signed-cert
    echo -n "admin" > /tmp/dashboard-password
    ceph dashboard set-login-credentials admin -i /tmp/dashboard-password
    rm /tmp/dashboard-password
    ```

2. Access the dashboard using the provided URL.

### Additional Tips

- Regularly check the cluster status with `ceph -s`.
- For advanced configurations, refer to the official Ceph documentation.
- Monitor logs and health alerts to maintain cluster integrity.

By following these steps, you should have a functioning Ceph cluster on your Rocky Linux nodes.