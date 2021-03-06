# Jenkins-series
This tutorial is used to teach you various pattern of jenkins setup with simple to advance docker . 

Here’s what we are going to learn this time:

        1. The High-Level Flowchart of CI Pipeline Using Jenkins
        2. Setting up CI Infrastructure-As-Code Using Docker
        3. Setting up a Custom Docker Image for Our Jenkins Master
        4. Configuring a “Dockerized” Build Agent for Compiling Our Code
        5. Writing the Docker File and Making It Self-Attaching Our Master
        6. Building an Image and Exposing the Docker Daemon Within the Agent
        7. Creating a Pipeline-As-A-Code Job in Jenkins
        8. Writing a Jenkins File
        9. Dynamic Build Versions
        10. Building the Application
        11. Running Tests and Publishing Reports in Jenkins
        12. Conclusion

Let’s dive right into it.

## The High-Level Flowchart of CI Pipeline Using Jenkins
        Let’s see what a “very” high-level view of our Continuous Integration (CI) Pipeline using Jenkins looks like:
        ![alt text](https://github.com/prateekdevisingh/jenkins-series/tree/master/images/HighLevelFlow.png)


        The flow diagram is easy to understand however, there are few things to note,

          - You may wonder as to where the code compilation and unit-test steps are? Well, the docker run step performs both the code               compile and - the application publish!
          - The dotted line marks the boundary of our CI tool, Jenkins, in this example
          - Having the docker push after a successful run of the “Integration Tests” ensures that only tested application is promoted to             the next   region, another very important rule of Continuous Integration (CI)

## Setting up CI Infrastructure-As-Code Using Docker
        Let’s get into the fun stuff now. Setting up our Jenkins infrastructure!

        Although there are many things to love about the Jenkins docker image there is something that is..… Let’s say “a little                 inconvenient” ?

        Let’s see what that little thing is.

        When we run the following docker command:
        [ docker run -d -p 8080:8080 -p 50000:50000 jenkins/jenkins ]

        check jenkins portal
        like : http://<ip>:8080

        Jenkins welcomes us with the “Setup Wizard Screen” which requires us to enter the “InitialAdminPassword” located under the               Jenkins_Home directory defaulted to /var/jenkins_home/secrets/initialAdminPassword. The password is displayed in the start-up           logs as well:
        ![alt text](https://github.com/prateekdevisingh/jenkins-series/tree/master/images/JenkinsScreen1.png)


## Setting up a Custom Docker Image for Our Jenkins Master
        Let’s look at the below Dockerfile:

        <your git path>/master/Dockerfile
        Note : open this Dockerfile on vi editor or any text editor 

        We start with the Jenkins/Jenkins base image and install the plugins that we require.

        Line# 12 is the run-time JVM parameter that needs to be passed in to disable the “Setup Wizard”

        Line# 15 and 16 is to provide the container with initial start-up scripts to set the Jenkins executors and for creating the             Jenkins admin user.

        Let’s build this image and keep it ready. We will get back to it right after we build our agent!er

        [docker build -t jenkins-master .]

## Configuring a “Dockerized” Build Agent for Compiling Our Code

        As for the Jenkins build agent, we will make it “auto-attaching” to the Jenkins master using JLNP.

        Here is what the agent’s Dockerfile looks like:

        <Your git path>/slave/Dockerfile
         Let’s look at some important steps in the above file,

        Line# 13 takes care of downloading and installing “docker-compose” in our build agent

        Line# 18 takes care of adding the magical start-up python script responsible for attaching to our master as a build agent!

        Now, who likes to go through a 90 line python script? Not me ?! To make things easier let’s look at this simple flowchart to             understand this!      
        ![alt text](https://github.com/prateekdevisingh/jenkins-series/tree/master/images/slave_python.png)
        
        The “wait for the master” logic is going to come in very handy when we wrap the master and slave into a docker-compose file. The         depends_on: tag in docker-compose doesn’t serve as well, as the jenkins master takes more time to be fully up and running than           what docker-compose estimates it to be. Thus, it’s added intelligence to our slave.

        Let’s build this container and warp our Jenkins infrastructure into a docker-compose file!
## Building an Image and Exposing the Docker Daemon to the Agent

        [docker build -t jenkins-slave .]
        
        <your git path>/docker-compose.ci.yml
        check this file and follow the steps below ,
        
        The Line# 17 is significant here as the mounted volume will help fix the docker in docker volume mount issue. Its covered well           in our previous article here

                        Great!

        Now its time to launch our Jenkins Infrastructure with the docker-compose up command
        [docker-compose -f .\docker-compose.ci.yml up]
        
        Here are the docker-compose logs for the container start-up:

        Jenkins container has started however, it’s not fully up and running. Thus, our wait logic coming in handy.
        And finally, our agent has successfully connected to our Jenkins master. We should be able to access the Jenkins web UI at port         8080 of our localhost!
        
        
