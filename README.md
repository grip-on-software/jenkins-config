# Jenkins configuration

This repository contains configuration for a Jenkins server as well as 
instructions for Jenkins build node setup. The configuration file can be used 
on Ubuntu to tweak the startup parameters of Jenkins. This configuration is 
tailored toward using Jenkins for building Grip on Software repositories and 
performing analysis/visualization, which requires further pipeline setup.

## Installation

See https://pkg.jenkins.io/debian-stable/ for more complete instructions for 
installing Jenkins on Ubuntu. We assume you have root privileges.

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
echo "deb https://pkg.jenkins.io/debian-stable binary/" > /etc/apt/sources.list.d/jenkins.list
apt-get update
apt-get install jenkins
```

Next, to set up the Jenkins configuration, clone this repository (for example 
in `/srv/jenkins-config`), and run the following commands:

```
mv /etc/default/jenkins /etc/default/jenkins.bak
ln -s /srv/jenkins-config/jenkins-config /etc/default/jenkins
```

When upgrading, use `apt-get install --only-upgrade jenkins`. APT might 
complain that the configuration file has been changed. Inspect the upstream 
changes through a diff, and keep the original symlink by choosing 'N' or 'O'. 
Apply any relevant changes from upstream to the configuration file and commit 
and push the updates.

## Configuration

Jenkins configuration is often specific to the environment it is in, such as 
authentication, email settings, connections to GitLab instance, data retention 
and access restrictions. For GROS, some specifics can be found here:

- Credential identifiers can be found in [credentials.md](credentials.md).
- Environment variables can be found in [environment.md](environment.md).
- Plugins used by pipelines can be found in [plugins.md](plugins.md).
- Tool configurations can be found in [tools.md](tools.md).

## Worker nodes

Documentation on setting up a secure, GPU-enabled worker node can be found in 
[jenkins-build-node.md](jenkins-build-node.md).
