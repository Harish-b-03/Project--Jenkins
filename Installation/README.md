# **Installation**
<br>

## **Docker Installation**

<br>
Jenkins pipelines work really well with Docker. Without Docker you need to install additional tools and add different agents to Jenkins. With Docker,

- there is no need to install additional tools, you just use images of these tools. Jenkins will download them from internet for you (Docker Hub).

- For each stage in the pipeline you can use a different image (i.e. tool). Essentially you get "micro Jenkins agents" which only exists temporary. Hence you do not need fixed agents anymore. This makes your Jenkins setup much more clean.

- Also, we can utilize all the advantages of Docker Containers

Still there are few disadvantages - priviledges.

<br>

## **Docker - Install using the repository**
<br>
Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

<br>


### **Set up the repository**

<br>

1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:

    ```
    sudo apt-get update

    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```
    <br>

2. Add Docker’s official GPG key:
    ```
    sudo mkdir -p /etc/apt/keyrings

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```

    <br>

3. Use the following command to set up the repository:

    ``` 
    echo \ 
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    (lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
<br>

### **Install Docker Engine**

<br>

1. Update the apt package index, and install the latest version of Docker Engine, containerd, and Docker Compose:

    ```
    sudo apt-get update
    
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```
2. To install a specific version of Docker Engine, list the available versions in the repo, then select and install:

    a. List the versions available in your repo:

    ```
    apt-cache madison docker-ce
    ```

    b. Install a specific version using the version string from the second column, for example, ```5:20.10.16~3-0~ubuntu-jammy```.

    ```
    sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io docker-compose-plugin
    ```

3. Verify that Docker Engine is installed correctly by running the ```hello-world``` image.
    ```
    sudo docker run hello-world
    ```

    This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

Docker Engine is installed and running. The ```docker``` group is created but no users are added to it. You need to use ```sudo``` to run Docker commands. Continue to Linux postinstall to allow non-privileged users to run Docker commands and for other optional configuration steps.

<br>

## **Jenkins Setup**

<br>

On Linux and MacOS:

1. Open up a terminal window.

2. Create a ```bridge network``` in Docker using the following ```docker network create``` command:

    ```
    docker network create jenkins
    ```

3. In order to execute Docker commands inside Jenkins nodes, download and run the ```docker:dind``` Docker image using the following ```docker run``` command:

    ```
    docker run \
        --name jenkins-docker \
        --rm \
        --detach \
        --privileged \
        --network jenkins \
        --network-alias docker \
        --env DOCKER_TLS_CERTDIR=/certs \
        --volume jenkins-docker-certs:/certs/client \
        --volume jenkins-data:/var/jenkins_home \
        --publish 2376:2376 \
        docker:dind \
        --storage-driver overlay2
    ```

4. Customise official Jenkins Docker image, by executing below two steps:
    
    a. Create Dockerfile with the following content:

    ```
    FROM jenkins/jenkins:2.346.1-jdk11
    USER root
    RUN apt-get update && apt-get install -y lsb-release
    RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
    https://download.docker.com/linux/debian/gpg
    RUN echo "deb [arch=$(dpkg --print-architecture) \
    signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
    https://download.docker.com/linux/debian \
    $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
    RUN apt-get update && apt-get install -y docker-ce-cli
    USER jenkins
    RUN jenkins-plugin-cli --plugins "blueocean:1.25.5 docker-workflow:1.28"
    ```
    b. Build a new docker image from this Dockerfile and assign the image a meaningful name, e.g. ```"myjenkins-blueocean:2.346.1-1"```:

    ```
    docker build -t myjenkins-blueocean:2.346.1-1 .
    ```

    Keep in mind that the process described above will automatically download the official Jenkins Docker image if this hasn’t been done before.

5. Run your own ```myjenkins-blueocean:2.346.1-1``` image as a container in Docker using the following ```docker run``` command:

    ```
    docker run \
        --name jenkins-blueocean \
        --restart=on-failure \
        --detach \
        --network jenkins \
        --env DOCKER_HOST=tcp://docker:2376 \
        --env DOCKER_CERT_PATH=/certs/client \
        --env DOCKER_TLS_VERIFY=1 \
        --publish 8080:8080 \
        --publish 50000:50000 \
        --volume jenkins-data:/var/jenkins_home \
        --volume jenkins-docker-certs:/certs/client:ro \
        myjenkins-blueocean:2.346.1-1 
    ```
<br>

### **Accessing the Docker container**

<br>

Access your Docker container through a terminal/command prompt using the ```docker exec ```command, you can add an option like ```--name jenkins-tutorial``` to the ```docker exec``` command. That will access the Jenkins Docker container named "jenkins-tutorial".

This means you could access your docker container (through a separate terminal/command prompt window) with a ```docker exec``` command like:

```
docker exec -it jenkins-blueocean bash
```


## **Unlocking Jenkins**


<br>

Refrences:

- [Stackoverflow - Advantages of running Jenkins in a Docker Container](https://stackoverflow.com/questions/44440164/what-are-the-advantages-of-running-jenkins-in-a-docker-container)
- [Docker Enginer Installation](https://docs.docker.com/engine/install/ubuntu/)
- [Jenkins Setup](https://www.jenkins.io/doc/book/installing/docker/)