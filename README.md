To setup the server:

1. install debian 10
danach ist apt-get verfügbar,

2. ssh key erzeugen
ssh key generieren:
ssh-keygen

3. auf jedem Knoten ssh server installieren und konfigurieren:
sudo apt-get install openssh-server ii.
sudo service ssh status
vim /etc/ssh/sshd_config
PermitRootLogin yesssh
/etc/init.d/ssh restart

4. public key auf allen Knoten kopieren
ssh-copy-id root@10.211.55.4

danach auf remote einloggen:

ssh root@10.211.55.4
-> exit
ssh-copy-id root@10.211.55.4
-> exit

5. Ansible auf lokalem Recher installieren

sudo -H pip3 install ansible
check:
ansible --version

6. Ansible hosts
hosts

7. Ansible: create non root users
initial.yml

ansible-playbook -i hosts initial.yml  -v --extra-vars "ansible_sudo_pass=OSVHyuuvis2021!"

8. Ansible: install dependencies (docker) and the 3 kubernetes components kubeadm, kubelet and kubectl

Achtung: auf master: apt-get install acl

ansible-playbook -i hosts kube-dependencies.yml  -v 

9. Create the master node

ansible-playbook -i hosts master.yml  -v 

10. create workers nodes

ansible-playbook -i hosts workers.yml -v

--------------------------------- if minio should be installed -----------------------

11. install nfs dependencies

ansible-playbook -i hosts nfs-dependencies.yml -v

12. install minio

ansible-playbook -i hosts minio.yml -v

http://10.211.55.4:31251/minio/login



Voraussetzungen debian für nfs:
sudo apt install nfs-common
Problems:

cat /usr/lib/systemd/system/etcd.service
change user etcd to root

systemctl status etcd.service

systemctl daemon-reload
systemctl restart etcd

create the mount directories: /tmp/m0 on the specific node

evtl. überspringen bis apt install nfs-kernel-server
nano /etc/default/nfs-common
NEED_STATD="no"
NEED_IDMAPD="yes"

nano /etc/default/nfs-kernel-server
RPCNFSDOPTS="-N 2 -N 3"
RPCMOUNTDOPTS="--manage-gids -N 2 -N 3"

sudo systemctl mask rpcbind.service
sudo systemctl mask rpcbind.socket

apt install nfs-kernel-server

vermutlich besser:
sudo apt-get install nfs-common nfs-kernel-server
cat /proc/fs/nfsd/versions

auf dem node, das die volumes bereitstellt (Host)

mkdir /tmp/m0
mkdir /tmp/m1
mkdir /tmp/m2
mkdir /tmp/m3

chmod 777 /tmp/m0
chmod 777 /tmp/m1
chmod 777 /tmp/m2
chmod 777 /tmp/m3

edit /etc/exports 

/tmp/m0       10.211.55.4(rw,sync,no_subtree_check)
/tmp/m0       10.211.55.9(rw,sync,no_subtree_check)
/tmp/m0       10.211.55.10(rw,sync,no_subtree_check)
/tmp/m1       10.211.55.4(rw,sync,no_subtree_check)
/tmp/m1       10.211.55.9(rw,sync,no_subtree_check)
/tmp/m1       10.211.55.10(rw,sync,no_subtree_check)
/tmp/m2       10.211.55.4(rw,sync,no_subtree_check)
/tmp/m2       10.211.55.9(rw,sync,no_subtree_check)
/tmp/m2       10.211.55.10(rw,sync,no_subtree_check)
/tmp/m3       10.211.55.4(rw,sync,no_subtree_check)
/tmp/m3       10.211.55.9(rw,sync,no_subtree_check)
/tmp/m3       10.211.55.10(rw,sync,no_subtree_check)

sudo systemctl restart nfs-kernel-server

to see all exports:
sudo exportfs -v

to see all mounts on a mchine:
df -h --output=source,target
try nfs mont:
mount -t nfs -o hard,nfsvers=3 10.211.55.10:/tmp/m3 /tmp/test -v
to unmount:
umount -f -l /tmp/test

docker info
shows:
Cgroup Driver: cgroupsfs
but it should be:
Cgroup Driver: systemd

In der Datei:
/usr/lib/systemd/system/docker.service
add --exec-opt....:
ExecStart=/usr/bin/dockerd --exec-opt native.cgroupdriver=systemd

and restart docker:

systemctl daemon-reload
systemctl restart docker

To delete configuration and/or data files of etcd and it's dependencies from Ubuntu Xenial then execute:
sudo apt-get autoremove --purge etcd

ssh jwewer@10.0.1.51

export KUBECONFIG=/etc/kubernetes/admin.conf
export KUBECONFIG=/Users/wewer/.kube/config/master/etc/kubernetes/admin.conf
sudo mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/

cd ~/.kube

sudo mv admin.conf config
sudo service kubelet restart

