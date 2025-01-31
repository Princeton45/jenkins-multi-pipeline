# My Journey Building a CI Pipeline with Jenkins for a Java Maven App

This documents my experience setting up a Continuous Integration (CI) pipeline using Jenkins for a Java application built with Maven. I created three different types of Jenkins jobs – Freestyle, Scripted Pipeline, and Multibranch Pipeline – each designed to automate the process of building the app, creating a Docker image, and pushing it to a private Docker Hub repository. 

The flow of each pipeline is as follows:

1. **Connect to Git Repository:** Each pipeline starts by connecting to my application's Git repository to pull the latest code.
2. **Build JAR:** Maven is used to build the Java application, resulting in a JAR file.
3. **Build Docker Image:** The JAR file and a Dockerfile are used to create a Docker image of the application.
4. **Push to Docker Hub:** Finally, the newly built Docker image is pushed to my private Docker Hub repository, making it ready for deployment.

![Diagram](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/diagram.jpg)


## Technologies I Used

*   **Jenkins**
*   **Docker** 
*   **Linux**
*   **Git**
*   **Java**
*   **Maven**

## My CI Pipeline Setup

Here's a breakdown of how I configured the pipeline:

### 1. Jenkins Server and Tooling

I started with a clean Jenkins server. I ensured that essential build tools like Maven and Node.js were installed in Jenkins, making them readily available for my pipeline jobs.

Maven was already available as a plugin through `Jenkins Tools`.

![maven-install](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/maven-install.png)

For Node.js and NPM, I installed those in the docker container that's running Jenkins

`docker exec -u 0 -it ae9bb3e63ec1 /bin/bash` <-- the `-u 0` runs the container as root.

`curl -sL https://deb.nodesource.com/setup_20.x -o nodesource_setup.sh`. 

The `nodesource_setup.sh ` is a script that contains the commands needed to install node.js and npm.

### 2. Freestyle Jenkins Job - Creation, Git & Docker connection

My first job type was a Freestyle job called `my-job`.

![freestyle](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/maven-job.png)

I configured it to pull the latest code from my Git repository, authenticating with a PAT token with least privilege.

![git-connection](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/git-connection.png)

I also made sure it will trigger only on changes to the `java-maven-app-starting-code` by creating an additional behaviour `Polling ignores commits in certain paths`.

![add-be](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/add-be.png)

I created another freestyle job called `java-maven-build` that takes the Java Maven app, runs tests and the builds a Jar file.

![java-maven-build](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/java-maven-build.png)

Then below you can see, when the job ran it created a `/target` folder locally in the Jenkins Docker container and then created the `.jar` file.

![jar](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/jar.png)

### 3. Make Docker available in Jenkins

I now have to make Docker available in Jenkins so that after building the application, we can convert it to a Docker image.

For this, I am going to mount the docker runtime directory thats on the VM inside the Jenkins container as a volume, which will make Docker available inside the container.

```bash
docker run -p 8080:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkins/jenkins:lts
```
Then we need to fetch the latest version of docker inside the container and set the permissions (also making sure that you exec into the container as root)

`curl https://get.docker.com/ > dockerinstall && chmod 777 dockerinstall && ./dockerinstall`

We should also change the permissions on the `docker.sock` file so the `jenkins` service account user can utilize it for docker

`chmod 666 /var/run/docker.sock`

### 4. Building the Docker image

Now that Docker is available in Jenkins, we can now add a step in the pipeline to build a Docker image using the built JAR artifact.

![docker](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/docker-step.png)

Remember we can use the `Execute Shell` option because we installed Docker inside the container (not as a plugin on Jenkins)

Now in the container, we can see the Docker image that was just built.

![built-image](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/built-image.png)

### 5. Push Image to Docker Hub

I edited the pipeline to push the created image into Docker Hub repository.

![dockerhub](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/dockerhub.png)

![dockerhub](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/dockerhub2.png)

### 6. Push Docker Image to Nexus Repository

I created a `docker.daemon` file in `/etc/docker` to add the `insecure-registries`
 parameter since our Nexus setup is configured to use HTTP and not HTTPS

![insecure](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/insecure11.png)

Now I pushed the Docker image to the Nexus Repository with this build step

![nexus](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/nexus.png)

Here it the docker that was pushed to my Nexus repository:

![nexus](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/nexus-push.png)


### 5. Pipeline Jenkins Job (Scripted Pipeline)

Next, I created a scripted Pipeline job which does the same as the freestyle job using a `Jenkinsfile`. This allowed me to define the entire pipeline as code. The pipeline checked out the code, built the JAR with Maven, built a Docker image, and finally pushed the image to my private Docker Hub repository.

`Jenkinsfile`
```groovy
def gv

pipeline {
    agent any
    tools {
        maven "maven-3.9"
    }
    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }
        stage("build jar") {
            steps {
                script {
                    gv.buildJar()
                }
            }
        }
        stage("build image") {
            steps {
                script {
                    gv.buildImage()
                    
                }
            }
        }
        stage("deploy") {            
            steps {
                script {
                    gv.deployApp()
                }
            }
        }                 
    }      
}
```

`script.groovy`
```groovy
def buildJar() {
    echo 'building the application...'
    sh "mvn package"
}

def buildImage() {
    echo "building the docker image..."
    withCredentials([usernamePassword(credentialsId: "docker-hub-repo", passwordVariable: "PASS", usernameVariable: "USER")]) {
    sh "docker build -t prince450/demo-app:jma-2.0 ."
    sh "echo $PASS | docker login -u $USER --password-stdin"
    sh "docker push prince450/demo-app:jma-2.0"
    }
}

def deployApp() {
    echo 'deploying the application...'
}

return this
```

The `Jenkinsfile` is cleaner because I'm just referencing the functions from the external `script.groovy` where all the logic is being defined.

![pipeline](https://github.com/Princeton45/jenkins-multi-pipeline/blob/main/images/pipeline.png)

### 6. Multibranch Pipeline Jenkins Job

Finally, I set up a Multibranch Pipeline job. This automatically detected branches in my Git repository and created corresponding pipelines for each branch. 

The pipeline will run the `Test`, `Build` and `Deploy` stages for pushes detected to the master branch and only run the `Test` stage for all other branches.

Best practice in a company is to have the feature branches run the test and optionally the build stage. Then development branches to run the test and build stages. Lastly, the main/master branch (production) would run `Test`, `Build` and `Deploy` stages


*   **Suggestions for Visuals:**
    *   **Picture 8:** The Jenkins dashboard showing the Multibranch Pipeline job with multiple branches (e.g., `main`, `develop`) and their build status.
    *   **Picture 9:** My Docker Hub repository showing the different image tags pushed by the Multibranch Pipeline, corresponding to different branches.

