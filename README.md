
## RHCE Task Solution

### Part 1: LVM Setup
```bash
fdisk -l                     # Check for available disks
fdisk /dev/sdb              # Create Disk Partitions

# Inside fdisk:
n
p
1
2048
+1GB
n
p
2
1955840
+1GB

fdisk -l
vgcreate -s 16M New_VG /dev/sdb1 /dev/sdb2
lvcreate -l 50 -n New_LV New_VG               # 50*16M total LV size
mkfs -t ext4 /dev/mapper/New_VG-New_LV
mkdir /mnt/data
mount /dev/mapper/New_VG-New_LV /mnt/data
echo "/dev/mapper/New_VG-New_LV /mnt/data ext4 defaults 0 0" >> /etc/fstab
```

### Part 2: User and Group Management
```bash
adduser user1
passwd user1                            # redhat
usermod -u 601 user1

echo "DenyUsers user1" >> /etc/ssh/sshd_config
systemctl restart sshd

groupadd TrainingGroup
usermod -aG TrainingGroup user1

useradd user2
useradd user3
groupadd admin
usermod -aG admin user2
usermod -aG admin user3
gpasswd admin                           # redhat

passwd user2                            # redhat
passwd user3                            # redhat

visudo                                  # Add:
user3 ALL=(ALL) NOPASSWD:ALL
```

### Part 3: SSH Connection (Assuming NAT Network)
```bash
# VM IPs: 192.168.1.4, 192.168.1.5
ssh-keygen
ssh-copy-id abed@192.168.1.4
ssh abed@192.168.1.4
```

### Part 4: ACL Permissions
```bash
getfacl /var/tmp/admin
setfacl -m u:user1:rwx /var/tmp/admin
setfacl -m u:user2:--- /var/tmp/admin
setfacl -m mask:rwx /var/tmp/admin
```

### Part 5: SELinux Enforce Mode
```bash
setenforce 1
getenforce
```

### Part 6: Run Script for 10 Minutes
```bash
touch script.sh
echo "
start_time=\$SECONDS
while (( SECONDS - start_time < 600 )); do
  echo hi >> test45.txt
  sleep 1
done
" >> script.sh

chmod +x script.sh
./script.sh &
jobs
fg          # Bring it to foreground
# or:
kill -9 PID
```

### Part 7: Local Yum Repository for Zabbix
```bash
yum install tmux httpd mysql createrepo -y
mkdir /var/www/html/zabbix-repo

wget -r -np -nH --cut-dirs=5 -R "index.html*" \
https://repo.zabbix.com/zabbix/4.4/rhel/7/x86_64/

createrepo .
chown -R apache:apache /var/www/html/zabbix-repo

firewall-cmd --add-service=http --permanent
firewall-cmd --reload

echo "
[zabbix-local]
name=Zabbix Local Repository
baseurl=http://<your_server_ip>/zabbix/
enabled=1
gpgcheck=0
" >> /etc/yum.repos.d/zabbix-local.repo

yum makecache
yum install zabbix-server zabbix-web zabbix-agent php
```

### Part 8: Firewall Rules
```bash
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --zone=public --add-port=80/tcp --permanent

firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.5" port port="22" protocol="tcp" reject' --permanent
```

### Part 9: Cron Job - Logged Users Timestamp
```bash
touch timestamp_users
echo "w >> timestamp_users" >> script2.sh
chmod +x script2.sh

crontab -e
# Add:
30 01 * * * /path/to/script2.sh
```

### Part 10: MariaDB + Student Table
```bash
yum install mariadb-server -y
iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
service iptables save
systemctl restart iptables
systemctl start mariadb
systemctl enable mariadb

mysql -u root

CREATE DATABASE studentdb;
CREATE USER 'abed'@'10.0.2.15' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON *.* TO 'abed'@'10.0.2.15' WITH GRANT OPTION;
FLUSH PRIVILEGES;
SET PASSWORD FOR 'abed'@'10.0.2.15' = PASSWORD('1234');
FLUSH PRIVILEGES;

-- Then login as abed:
mysql -u abed -p studentdb

CREATE TABLE students (
  student_number VARCHAR(10) PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50),
  program_enrolled VARCHAR(50),
  graduation_year INT
);

INSERT INTO students VALUES
('110-001', 'Allen', 'Brown', 'Mechanical', 2017),
('110-002', 'David', 'Brown', 'Mechanical', 2017),
('110-003', 'Mary', 'Green', 'Electrical', 2018),
('110-004', 'Dennis', 'Green', 'Electrical', 2018),
('110-005', 'Joseph', 'Black', 'Computer Science', 2020),
('110-006', 'Dennis', 'Black', 'Computer Science', 2020),
('110-007', 'Ritchie', 'Salt', 'Computer Science', 2020),
('110-008', 'Robert', 'Salt', 'Computer Science', 2020),
('110-009', 'David', 'Suzuki', 'Electrical', 2018),
('110-010', 'Mary', 'Chen', 'Mechanical', 2017);
```
