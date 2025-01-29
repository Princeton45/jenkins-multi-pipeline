# My Journey Building a CI Pipeline with Jenkins for a Java Maven App

This documents my experience setting up a Continuous Integration (CI) pipeline using Jenkins for a Java application built with Maven. I created three different types of Jenkins jobs – Freestyle, Scripted Pipeline, and Multibranch Pipeline – each designed to automate the process of building the app, creating a Docker image, and pushing it to a private Docker Hub repository. The flow of each pipeline is as follows:

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

I started with a clean Jenkins server. I ensured that essential build tools like Maven and Node.js were installed directly within Jenkins, making them readily available for my pipeline jobs.

*   **Suggestions for Visuals:**
    *   **Picture 1:** A screenshot of the Jenkins "Global Tool Configuration" page showing Maven and Node.js configured.

### 2. Docker Integration

To enable Docker operations within my pipeline, I installed Docker on the Jenkins server and configured the `jenkins` user to run Docker commands seamlessly.

*   **Suggestions for Visuals:**
    *   **Picture 2:** My terminal showing the successful output of `docker info` run as the `jenkins` user, confirming Docker is accessible to Jenkins.

### 3. Git Repository Credentials

I created credentials within Jenkins to allow it to securely connect to and pull code from my private Git repository.

*   **Suggestions for Visuals:**
    *   **Picture 3:** The Jenkins credentials page showing the stored credentials for my Git repository.

### 4. Freestyle Jenkins Job

My first job type was a Freestyle project. I configured it to pull the latest code from my Git repository and use Maven to build the Java application, creating the JAR file.

*   **Suggestions for Visuals:**
    *   **Picture 4:** A screenshot of the successful build history of the Freestyle job in Jenkins, showing green checkmarks for each build.
    *   **Picture 5:** The Jenkins console output for a successful Freestyle build, highlighting the `BUILD SUCCESS` message from Maven.

### 5. Pipeline Jenkins Job (Scripted Pipeline)

Next, I created a scripted Pipeline job using a `Jenkinsfile`. This allowed me to define the entire pipeline as code. The pipeline checked out the code, built the JAR with Maven, built a Docker image, and finally pushed the image to my private Docker Hub repository.

*   **Suggestions for Visuals:**
    *   **Picture 6:** A screenshot of my `Jenkinsfile` in a code editor, showcasing the pipeline stages.
    *   **Picture 7:** The "Pipeline Steps" view in Jenkins for a successful pipeline run, showing each stage (Checkout, Build, Docker Build, Docker Push) completed.

### 6. Multibranch Pipeline Jenkins Job

Finally, I set up a Multibranch Pipeline job. This automatically detected branches in my Git repository and created corresponding pipelines for each branch. Each branch pipeline followed the same steps as the scripted pipeline: checkout, build JAR, build Docker image, and push to Docker Hub.

*   **Suggestions for Visuals:**
    *   **Picture 8:** The Jenkins dashboard showing the Multibranch Pipeline job with multiple branches (e.g., `main`, `develop`) and their build status.
    *   **Picture 9:** My Docker Hub repository showing the different image tags pushed by the Multibranch Pipeline, corresponding to different branches.

## Conclusion

And that's a wrap! I now have a fully automated CI pipeline powered by Jenkins that handles building, containerizing, and deploying my Java application. I explored different job types, each with its strengths, and ended up with a robust solution that fits my needs. This setup saves me tons of time and ensures that my application is always ready to be deployed. I hope my journey helps you in setting up your own CI pipelines! Feel free to experiment and adapt these concepts to your projects. Happy coding!