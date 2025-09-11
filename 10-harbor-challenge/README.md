# Challenge: Install Harbor

## Description:

Convert the bash script below into an Ansible playbook. Then run and install Harbor on server 2. 

```bash
wget -O harbor.tgz https://github.com/goharbor/harbor/releases/download/v2.5.6/harbor-online-installer-v2.5.6.tgz
tar xf harbor.tgz
cp harbor/harbor.yml.tmpl harbor/harbor.yml
sed -i "s/hostname: .*/hostname: harbor.dev.purelogicit.ca/g" harbor/harbor.yml
sed -i "s/  certificate: .*/  certificate: \/data\/certs\/harbor.crt/g" harbor/harbor.yml
sed -i "s/  private_key: .*/  private_key: \/data\/certs\/harbor.key/g" harbor/harbor.yml
cd harbor && sudo./install.sh --with-trivy
```