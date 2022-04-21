# Setup Jenkins GPU/CPU build node

This is an example build node setup for a Jenkins agent which is able to handle 
jobs that require, for example, an NVIDIA GPU with CUDA for machine learning, 
Docker and docker-compose for building and testing images, Ant for Java builds, 
Python for analysis, cleanup and various other build scripts, so that it can 
function as a multipurpose agent that builds many GROS projects.

We assume a Ubuntu setup; for other Linux distributions the install procedures
will be different but may be useful hints.

## Initial dependencies

As root, check if the following packages are installed and if not, install them:

```
dpkg -l ca-certificates software-properties-common
apt-get install ca-certificates software-properties-common
```

Also, install a flavor of `python3`, `python3-dev`, `python3-distutils` and
`python3-pip` (we have used Python 3.6 and Python 3.8 before, the exact version
may differ and you can try `python3.x` where `x` is e.g. 6 or 8 instead), as
well as `openjdk-8-jre-headless` and `openjdk-8-jdk-headless`, respectively for
Pylint, SonarScanner and Ant compatibility.

## Filesystem

Ensure you have enough space to install Docker, as well as for Jenkins to
manage a local workspace. If necessary, you can create new volumes for the
Docker storage path and the Jenkins workspace.

Example of a separate Docker storage path using LVM, assuming an already
existing LVM setup with VGs where `vg1` has space left (always check `pvs`,
`vgs`, and `lvs` for space):

```
lvcreate -n docker -L 110G vg1
mkfs.ext4 /dev/mapper/vg1-docker
mount -t ext4 /dev/mapper/vg1-docker /var/lib/docker
```

The last command should ideally be replaced with an addition to `/etc/fstab`
with the proper settings, followed by `mount -a`. The following settings work:

```
/dev/mapper/vg1-docker /var/lib/docker        ext4    defaults        0       2
```

Example of an encrypted volume on a full physical disk that can contain the
Jenkins workspace, which will process data (even though this data may already
have been made pseudonymous through encryption, this option may be taken if
organizations require this as part of a data processing agreement) and have
credentials in its workspace/home directory, requires installation of `haveged`
and `cryptsetup`:

```
parted -a optimal /dev/sda mklabel msdos
parted -a optimal /dev/sda mkpart primary ext4 0% 100%
parted -a optimal /dev/sda name 1 crypto-luks
haveged -n 0 | dd of=/dev/sda1 # Make data hard to find
cryptsetup -v -s 512 luksFormat --use-random /dev/sda1 # Set a passphrase
cryptsetup luksOpen /dev/sda1 gros # Enter passphrase
pvcreate /dev/mapper/gros
vgcreate vg2 /dev/mapper/gros
lvcreate -l 100%FREE vg2 -n gros
mkfs.ext4 /dev/mapper/vg2-gros
mkdir /gros
mount -t ext4 /dev/mapper/vg2-gros /gros
```

Because the encrypted volume requires a passphrase to be unlocked, it cannot be
mounted at (re)boot, which would mean the Jenkins agent cannot start immediately
(but this is not a big problem because the Jenkins server will repeatedly try
to start when possible, when configured to do so there). In order to properly
mount the volume after each boot, perform the following steps manually:

```
cryptsetup luksOpen /dev/sda1 gros # Enter passphrase
mount /dev/mapper/vg2-gros /gros
```

## Docker

Setup docker repository:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
cat /etc/apt/sources.list
apt-get install docker-ce docker-ce-cli containerd.io
```

Setup nvidia-docker repository:

```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID); curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | tee /etc/apt/sources.list.d/nvidia-docker.list
apt-get update
apt-get install nvidia-docker2
```

To use the agent for builds of the `visualization-site` project, Docker Compose
is a requirement. As per https://docs.docker.com/compose/install/ you can
install via curl or with pip: `pip3 install docker-compose`.

## Workspace

In the root directory (e.g. `/` or a path specific for this setup like `/gros`),
set up the Jenkins user/workspace:

```
adduser --home var/lib/jenkins --disabled-password jenkins
adduser jenkins docker
rm var/lib/jenkins/examples.desktop
```

Ensure root directory is readable by Jenkins, otherwise:

```
chmod g+rx .
chown root:jenkins .
```

Add the node to the firewall of the Jenkins server to accept HTTPS/SSH/Docker
registry requests from the node's address.

Set up SSH logins and Jenkins agent:

- Run `su - jenkins`
- Run `ssh-keygen -t rsa -C "jenkins agent" -b 4096`
- Use default path
- Create a passphrase and store it in a credentials keychain, enter it twice
- Run `cp .ssh/id_rsa.pub .ssh/authorized_keys`
- Create a New Node on Jenkins, enter name of machine and select "Persistent".
  Then fill in "Description" (capitalized name plus a purpose in parentheses),
  "# of executors" (# of GPUs), "Remote root directory" (root directory plus
  `var/lib/jenkins`, i.e. `/gros/var/lib/jenkins` for our example setup)
  "Labels" (at least `gpu`, any others space separated, for visualization
  integration build add `docker-compose`), "Launch method -> Host" (FQDN of
  the machine where the agent runs). Use "Credentials -> Add -> Jenkins",
  choose "SSH Username with private key", fill in "ID" (hostname), "Description"
  (hostname plus "Jenkins SSH key"), "Username" (jenkins), "Private Key"
  (copy the contents of `id_rsa`) and "Passphrase" (from keychain), and finally
  add any applicable schedules and other properties for the agent.

Set up Docker login:

- Place HTTPS certificate from the Jenkins server (called `jenkins.example` in 
  the next steps, replace it with the actual internal/external domain name) in
  `/usr/local/share/ca-certicficates` and run `update-ca-certificates`, and
  also in `/etc/docker/certs.d/jenkins.example/ca.crt` (create directories first
  with `mkdir -p /etc/docker/certs.d/jenkins.example`)
- Either just do `docker login https://jenkins.example` (which will store the
  password unencrypted and remain giving warnings) or set up a credential helper
  (possibly only needed temporarily in order to set up the pass store). E.g.,
  with a GPG key:

```
gpg --batch --pinentry loopback --passphrase '' --quick-gen-key \
	"Jenkins Agent <jenkins-agent@jenkins.example>"
pass init <keyid> # Fill in key ID from output of GPG generation
docker-credential-pass store
```
