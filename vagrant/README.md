    1. install virtualbox 7.0.20
    2. install vagrant 2.4.0
    3. install vagrant 'vagrant-vbguest' pluging with: `vagrant plugin install vagrant-vbguest`
    4. run following commands
    ```bash
    git clone git@github.com:Sajjadhz/ceph_guide.git
    cd ceph_guide/vagrant
    chmod +x ceph_provision.sh
    vagrant up
    ```