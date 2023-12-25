# Hardware requirement:
```
hệ điều hành ubuntu  
ram = 4GB  
CPU: càng nhiều core càng tốt, ở đây sử dụng 4 core làm lab  
Tắt SElinux  
```
# Update package and set sync time:
```
apt update
apt upgrade  
timedatectl set-timzone [timezone]
```
# Install openvpn access server:  
```
apt update && apt -y install ca-certificates wget net-tools gnupg
wget https://as-repository.openvpn.net/as-repo-public.asc -qO /etc/apt/trusted.gpg.d/as-repository.asc
echo "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/as-repository.asc] http://as-repository.openvpn.net/as/debian jammy main">/etc/apt/sources.list.d/openvpn-as-repo.list
apt update && apt -y install openvpn-as
```
**Note:** remember account login after setup
```
+++++++++++++++++++++++++++++++++++++++++++++++ 
Access Server 2.11.3 has been successfully installed in /usr/local/openvpn_as
Configuration log file has been written to /usr/local/openvpn_as/init.log

Access Server Web UIs are available here:
Admin UI: https://192.168.102.130:943/admin
Client UI: https://192.168.102.130:943 
Login as "openvpn" with "RandomPassword" to continue <---- **Here**
(password can be changed on Admin UI)
+++++++++++++++++++++++++++++++++++++++++++++++
```
