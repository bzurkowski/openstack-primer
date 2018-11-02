# Installing OpenStack with Kolla Ansible

The following guide describes installation of All-in-One OpenStack using Kolla Ansible.

## Host requirements

The host machine must satisfy the minimum requirements:

* 2 network interfaces
* 8GB main memory
* 40GB disk space

## Dependencies

Install and upgrade pip:

```bash
$ apt-get update
$ apt-get install python-pip
$ pip install -U pip
```

Install dependencies:

```bash
$ apt-get install python-dev \
                  libffi-dev gcc \
                  libssl-dev \
                  python-selinux \
                  python-setuptools
```

### Ansible

Install and upgrade Ansible:

```bash
$ apt-get install ansible
$ pip install -U ansible
```

Modify Ansible configuration in `/etc/ansible/ansible.cfg`:

```ini
# /etc/ansible/ansible.cfg
[defaults]
host_key_checking=False
pipelining=True
forks=100
```

### Kolla Ansible

Install Kolla Ansible:

```bash
$ pip install kolla-ansible
```

Copy deployment confiugration and inventories:

```
$ cp -r /usr/local/share/kolla-ansible/etc_examples/kolla /etc/
$ cp /usr/local/share/kolla-ansible/ansible/inventory/* .
```

Clone repository:

```bash
$ git clone https://github.com/openstack/kolla-ansible
```

### Docker

Install Docker following the [instructions](docker.md).

Create Docker drop-in unit file:

```bash
$ mkdir -p /etc/systemd/system/docker.service.d
```

```ini
# /etc/systemd/system/docker.service.d/kolla.conf
[Service]
MountFlags=shared
```

Reload systemd daemon and enable Docker service:

```bash
$ systemctl daemon-reload
$ systemctl restart docker
$ systemctl enable docker
```

## Prepare deployment

Generate passwords:

```bash
$ kolla-genpwd
```

Edit `/etc/kolla/globals.yml`:

```yml
# /etc/kolla/globals.yml
kolla_base_distro: "centos"
kolla_install_type: "source"
openstack_release: "rocky"
kolla_internal_vip_address: "<IP address assigned to network_interface>"
network_interface: "eth0"
neutron_external_interface: "eth1"
enable_cinder: "yes"
enable_cinder_backup: "no"
enable_cinder_backend_lvm: "yes"
enable_haproxy: "no"
enable_horizon: "yes"
enable_horizon_trove: "{{ enable_trove | bool }}"
enable_trove: "yes"
```

### Cinder

Add `configfs` to `/etc/modules`:

```bash
$ echo "configfs" >> /etc/modules
```

Rebuild initramfs:

```bash
$ update-initramfs -u
```

Stop and disable iscsi:

```
$ systemctl stop open-iscsi
$ systemctl disable open-iscsi

$ systemctl stop iscsid
$ systemctl disable iscsid
```

Reboot server:

```bash
$ reboot
```

Make sure configfs gets mounted during a server boot up process:

```bash
$ mount -t configfs /etc/rc.local /sys/kernel/config
mount: /etc/rc.local is already mounted or /sys/kernel/config busy
```

Create LVM volume group using loopback mounted file:

```bash
$ free_device=$(losetup -f)
$ fallocate -l 20G /var/lib/cinder_data.img
$ losetup $free_device /var/lib/cinder_data.img
$ pvcreate $free_device
$ vgcreate cinder-volumes $free_device
```

## Run deployment

Execute prechecks:

```
$ kolla-ansible -i ./all-in-one prechecks
```

Deploy:

```
$ kolla-ansible -i ./all-in-one deploy
```

## Post deployment

Install OpenStack clients:

```bash
$ pip install python-openstackclient \
              python-troveclient \
              python-glanceclient \
              python-neutronclient
```

Generate credentials file:

```
$ kolla-ansible -i ./all-in-one post-deploy
```

Source credentials:

```bash
$ . /etc/kolla/admin-openrc.sh
```

Source credentials on each session login:

```bash
$ echo ". /etc/kolla/admin-openrc.sh" >> ~/.bashrc
```

Create example networks, images and flavors:

```bash
$ . /usr/local/share/kolla-ansible/init-runonce
```
