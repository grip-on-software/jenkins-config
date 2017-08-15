# Jenkins configuration

This configuration file can be used on Ubuntu to tweak the startup parameters 
of Jenkins.

## Installation

See https://pkg.jenkins.io/debian-stable/ for complete instructions for
installing Jenkins on Ubuntu. We assume you have root privileges.

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
echo "deb https://pkg.jenkins.io/debian-stable binary/" > /etc/apt/sources.list.d/jenkins.list
apt-get update
apt-get install jenkins
```

Next, to set up the Jenkins configuration:

```
git clone https://git.liacs.nl/gros/jenkins-config.git
mv /etc/default/jenkins /etc/default/jenkins.bak
ln -s /srv/jenkins-config/jenkins-config /etc/default/jenkins
```

When upgrading, use `apt-get install --only-upgrade jenkins`. APT is going to
complain that the configuration file has been changed. Inspect the upstream
changes through a diff, and keep the original symlink by choosing 'N' or 'O'.
Apply any relevant changes from upstream to the configuration file and commit
and push the updates.
