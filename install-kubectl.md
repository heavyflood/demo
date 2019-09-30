# install kubectl for client


**1. Download the latest release with the command**

    curl -LO https://storage.googleapis.com/kubernetes-release/release/curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt/bin/linux/amd64/kubectl

**> to download Specific version**

    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.16.0/bin/linux/amd64/kubectl

**2. Make the kubectl binary executable.**

    chmod +x ./kubectl

**3. Move the binary in to your PATH.**

    mv ./kubectl /usr/local/bin/kubectl

**4. Test to ensure the version you installed is up-to-date:

    kubectl version**

**5. install apt-get on the container**

    rpm -qa | grep rpmforge-release
