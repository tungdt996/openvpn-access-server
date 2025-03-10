# Hardware requirement:
```
OS: ubuntu  
ram = 4GB  
CPU
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
# Unlock license
**License in pyovpn-2.0-py3.8.egg (with openvpnas v2.12.1) (path:/usr/local/openvpn_as/lib/python)**
### Stop server:
```systemctl stop openvpnas```
### cd to folder and backup mkdir unlock_license:
```
cd /usr/local/openvpn_as/lib/python
mkdir unlock_license
mv pyovpn-2.0-py3.8.egg pyovpn-2.0-py3.8.egg_bak
cp -rp pyovpn-2.0-py3.8.egg_bak unlock_license/pyovpn-2.0-py3.8.zip
cd unlock_license/
unzip pyovpn-2.0-py3.8.zip
cd pyovpn/lic/
pip3 install uncompyle6
uncompyle6 uprop.pyc > uprop.py
vi uprop.py
```
**file uprop.py has the following content:**
```
    def figure(self, licdict):
        proplist = set(('concurrent_connections', ))
        good = set()
        ret = None
        if licdict:
            for key, props in list(licdict.items()):
                if 'quota_properties' not in props:
                    print('License Manager: key %s is missing usage properties' % key)
                else:
                    proplist.update(props['quota_properties'].split(','))
                    good.add(key)

        for prop in proplist:
            v_agg = 0
            v_nonagg = 0
            if licdict:
                for key, props in list(licdict.items()):
                    if key in good:
                        if prop in props:
                            try:
                                nonagg = int(props[prop])
                            except:
                                raise Passthru('license property %s (%s)' % (prop, props.get(prop).__repr__()))

                            v_nonagg = max(v_nonagg, nonagg)
                            prop_agg = '%s_aggregated' % prop
                            agg = 0
                            if prop_agg in props:
                                try:
                                    agg = int(props[prop_agg])
                                except:
                                    raise Passthru('aggregated license property %s (%s)' % (
                                     prop_agg, props.get(prop_agg).__repr__()))

                                v_agg += agg
                        if DEBUG:
                            print('PROP=%s KEY=%s agg=%d(%d) nonagg=%d(%d)' % (
                             prop, key, agg, v_agg, nonagg, v_nonagg))

            apc = self._apc()
            v_agg += apc
            if ret == None:
                ret = {}
            ret[prop] = max(v_agg + v_nonagg, bool('v_agg') + bool('v_nonagg'))
            ret['apc'] = bool(apc)
            if DEBUG:
                print("ret['%s'] = v_agg(%d) + v_nonagg(%d)" % (prop, v_agg, v_nonagg))

        return ret
```
**add ret[‘concurrent_connections’] = 2048 before return ret**
**The file will now look like this:**
```
    def figure(self, licdict):
        proplist = set(('concurrent_connections', ))
        good = set()
        ret = None
        if licdict:
            for key, props in list(licdict.items()):
                if 'quota_properties' not in props:
                    print('License Manager: key %s is missing usage properties' % key)
                else:
                    proplist.update(props['quota_properties'].split(','))
                    good.add(key)
 
        for prop in proplist:
            v_agg = 0
            v_nonagg = 0
            if licdict:
                for key, props in list(licdict.items()):
                    if key in good:
                        if prop in props:
                            try:
                                nonagg = int(props[prop])
                            except:
                                raise Passthru('license property %s (%s)' % (prop, props.get(prop).__repr__()))
 
                            v_nonagg = max(v_nonagg, nonagg)
                            prop_agg = '%s_aggregated' % prop
                            agg = 0
                            if prop_agg in props:
                                try:
                                    agg = int(props[prop_agg])
                                except:
                                    raise Passthru('aggregated license property %s (%s)' % (
                                     prop_agg, props.get(prop_agg).__repr__()))
 
                                v_agg += agg
                        if DEBUG:
                            print('PROP=%s KEY=%s agg=%d(%d) nonagg=%d(%d)' % (
                             prop, key, agg, v_agg, nonagg, v_nonagg))
 
            apc = self._apc()
            v_agg += apc
            if ret == None:
                ret = {}
            ret[prop] = max(v_agg + v_nonagg, bool('v_agg') + bool('v_nonagg'))
            ret['apc'] = bool(apc)
            if DEBUG:
                print("ret['%s'] = v_agg(%d) + v_nonagg(%d)" % (prop, v_agg, v_nonagg))
                
        ret['concurrent_connections'] = 2048
        return ret
```
###package files:
```
rm -f uprop.pyc
python3 -O -m compileall uprop.py && mv __pycache__/uprop.cpython-38.opt-1.pyc uprop.pyc
zip -r pyovpn-2.0-py3.8_cracked.zip common EGG-INFO pyovpn
mv pyovpn-2.0-py3.8_cracked.zip pyovpn-2.0-py3.8.egg
mv pyovpn-2.0-py3.8.egg ../ && cd ..
```
###clear python cache:
```
cd /usr/local/openvpn_as/lib/python
rm -rf __pycache__/*
```
###recreate profile openvpn
```
cd /usr/local/openvpn_as/bin  
./ovpn-init
```
