<h1>Install Docker & Docker Compose - Centos 7</h1>

<h2>Step 1 — Install Docker</h2>

**Install needed packages:**

    $ sudo yum install -y yum-utils device-mapper-persistent-data lvm2

**Configure the docker-ce repo:**

    $ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

**Install docker-ce:**

    $ sudo yum install docker-ce

**Add your user to the docker group with the following command.**

    $ sudo usermod -aG docker $(whoami)

**Set Docker to start automatically at boot time:**

    $ sudo systemctl enable docker.service

**Finally, start the Docker service:**

    $ sudo systemctl start docker.service

<h2>Step 2 — Install Docker Compose</h2>

    $ curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

    $ chmod +x /usr/local/bin/docker-compose

**To verify a successful Docker Compose installation, run:**

    $ docker-compose version
