# Script Guide to Create Octavia Certificate 

source: [docs](https://github.com/99cloud/lab-openstack/blob/master/doc/installation-kolla-multinode.md#%E9%85%8D%E7%BD%AE-octavia)

## Creating the Certificate Authorities

**Run All commands below only on Controller Node or Deployer Node**

**Run commands with Regular users (non-root)**

### 1. Clone Octavia Repository

```
cd ~
git clone https://opendev.org/openstack/octavia -b stable/ussuri
cd octavia/bin
```

### 2. Octavia Ca Password

Grep the octavia ca pass from passwords.yml

```
grep octavia_ca_password /etc/kolla/passwords.yml 

octavia_ca_password: 9U79wmh0tksD0oL8QMUFZRmrvBWV6MgiLVmVtBMv
```


### 3. Edit Script

```
sed -i 's/not-secure-passphrase/<octavia_ca_password>/g' create_single_CA_intermediate_CA.sh
```

### 4. Run Script

```
./create_single_CA_intermediate_CA.sh openssl.cnf
```

### 5. Copy Certificate to kolla config directory

```
cd single_ca/etc/octavia/certs/
sudo mkdir -p /etc/kolla/config/octavia
sudo chown -R $USER:$USER /etc/kolla/config
cp * /etc/kolla/config/octavia
cd ~
```