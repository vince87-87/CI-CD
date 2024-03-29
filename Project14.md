# Experience Continuous Integration with Jenkins | Ansible | Artifactory | SonarQube | PHP

Set Up

This project is partly a continuation of your Ansible work, so simply add and subtract based on the new setup in this project. It will require a lot of servers to simulate all the different environments from dev/ci all the way to production. This will be quite a lot of servers altogether (But you don’t have to create them all at once. Only create servers required for an environment you are working with at the moment. For example, when doing deployments for development, do not create servers for integration, pentest, or production yet).

To minimize the cost of cloud servers, you don not have to create all the servers at once, simply spin up a minimal server set up as you progress through the project implementation and have reached a need for more.

To get started, we will focus on these environments initially.

Ci

Dev

Pentest

Both SIT - For System Integration Testing and UAT - User Acceptance Testing do not require a lot of extra installation or configuration. They are basically the webservers holding our applications. But Pentest - For Penetration testing is where we will conduct security related tests, so some other tools and specific configurations will be needed. In some cases, it will also be used for Performance and Load testing. Otherwise, that can also be a separate environment on its own. It all depends on decisions made by the company and the team running the show.

What we want to achieve, is having Nginx to serve as a reverse proxy for our sites and tools. Each environment setup is represented in the below table and diagrams.

![image](https://user-images.githubusercontent.com/49937302/123425276-87157900-d5f4-11eb-98d0-876c2fce34c1.png)

# CI-Environment

![image](https://user-images.githubusercontent.com/49937302/123425354-a3191a80-d5f4-11eb-825e-904663010b67.png)

# Other Environments from Lower To Higher

![image](https://user-images.githubusercontent.com/49937302/123425430-b88e4480-d5f4-11eb-8df5-a070558ef525.png)

# DNS requirements

Make DNS entries to create a subdomain for each environment. Assuming your main domain is darey.io

You should have a subdomains list like this:

![image](https://user-images.githubusercontent.com/49937302/123425652-fbe8b300-d5f4-11eb-87c4-fb1f0583c71e.png)

# Ansible Inventory should look like this

![image](https://user-images.githubusercontent.com/49937302/123425938-51bd5b00-d5f5-11eb-8ea5-bc065d956b2b.png)

ci inventory file



dev Inventory file


pentest inventory file


# Observations:

You will notice that in the pentest inventory file, we have introduced a new concept pentest:children This is because, we want to have a group called pentest which covers Ansible execution against both pentest-todo and pentest-tooling simultaneously. But at the same time, we want the flexibility to run specific Ansible tasks against an individual group.

The db group has a slightly different configuration. It uses a RedHat/Centos Linux distro. Others are based on Ubuntu (in this case user is ubuntu). Therefore, the user required for connectivity and path to python interpreter are different. If all your environment is based on Ubuntu, you may not need this kind of set up. Totally up to you how you want to do this. Whatever works for you is absolutely fine in this scenario.
This makes us to introduce another Ansible concept called group_vars. With group vars, we can declare and set variables for each group of servers created in the inventory file.

For example, If there are variables we need to be common between both pentest-todo and pentest-tooling, rather than setting these variables in many places, we can simply use the group_vars for pentest. Since in the inventory file it has been created as pentest:children Ansible recognizes this and simply applies that variable to both children.

# Ansible Roles for CI Environment

Add two more roles to ansible:

SonarQube 

Artifactory

![image](https://user-images.githubusercontent.com/49937302/123434680-5424b280-d5ff-11eb-92f2-573a8b657b33.png)

# Why do we need SonarQube?

SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality, it is used to perform automatic reviews with static analysis of code to 

detect bugs, code smells, and security vulnerabilities. Watch a short description here. There is a lot more hands on work ahead with SonarQube and Jenkins. So, the purpose of 

SonarQube will be clearer to you very soon.

# Why do we need Artifactory?

Artifactory is a product by JFrog that serves as a binary repository manager. The binary repository is a natural extension to the source code repository, in that the outcome of 

your build process is stored. It can be used for certain other automation, but we will it strictly to manage our build artifacts.

Watch a short description here Focus more on the first 10.08 mins

# Configuring Ansible For Jenkins Deployment

In previous projects, you have been launching Ansible commands manually from a CLI. Now, with Jenkins, we will start running Ansible from Jenkins UI.

To do this,

Navigate to Jenkins URL

Install & Open Blue Ocean Jenkins Plugin

![image](https://user-images.githubusercontent.com/49937302/123436841-a49d0f80-d601-11eb-8bec-995ccc1cbb44.png)

![image](https://user-images.githubusercontent.com/49937302/123437277-15442c00-d602-11eb-9f1a-ed331675496d.png)

![image](https://user-images.githubusercontent.com/49937302/123437715-8e438380-d602-11eb-9b65-3ab23da5b025.png)

Connect Jenkins with GitHub

![image](https://user-images.githubusercontent.com/49937302/123438378-36594c80-d603-11eb-8fca-2e584f41601b.png)

Login to GitHub & Generate an Access Token

![image](https://user-images.githubusercontent.com/49937302/123438434-44a76880-d603-11eb-8005-9e5e3a682afb.png)

![image](https://user-images.githubusercontent.com/49937302/123438508-5557de80-d603-11eb-9fc6-cb2ca5c614bf.png)

Copy Access Token

![image](https://user-images.githubusercontent.com/49937302/123438628-7ae4e800-d603-11eb-90ac-0715f8bbd24e.png)

Paste the token and connect

![image](https://user-images.githubusercontent.com/49937302/123438977-d911cb00-d603-11eb-8fe7-c3b1fe81e3cf.png)

Create a new pipeline

![image](https://user-images.githubusercontent.com/49937302/123442079-1af04080-d607-11eb-9390-e98f94fec594.png)

At this point you may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some guidance to create one. But we do not need that. We will rather create one ourselves. So, click on Administration to exit the Blue Ocean console.

![image](https://user-images.githubusercontent.com/49937302/123445495-84be1980-d60a-11eb-94a6-e5b6df32b3c2.png)

Here is our newly created pipeline. It takes the name of your GitHub repository.

![image](https://user-images.githubusercontent.com/49937302/123445872-e67e8380-d60a-11eb-84f2-c5cebf404f0f.png)

# Let us create our Jenkinsfile

Inside the Ansible project, create a new directory deploy and start a new file Jenkinsfile inside the directory.

![image](https://user-images.githubusercontent.com/49937302/123446696-b4b9ec80-d60b-11eb-9e5a-b0b6c68f8321.png)

Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage

![image](https://user-images.githubusercontent.com/49937302/123447006-019dc300-d60c-11eb-982a-07151aab02cf.png)

Now go back into the Ansible pipeline in Jenkins, and select configure

![image](https://user-images.githubusercontent.com/49937302/123513813-e097a980-d6c1-11eb-8087-12200920db31.png)

Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile

![image](https://user-images.githubusercontent.com/49937302/123514064-45073880-d6c3-11eb-9899-7a6200b0c119.png)

This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.

![image](https://user-images.githubusercontent.com/49937302/123514320-03778d00-d6c5-11eb-81db-a81d70c7cf9c.png)

To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.

Click on Blue Ocean

Select your project

Click on the play button against the branch 

![image](https://user-images.githubusercontent.com/49937302/123514461-bea02600-d6c5-11eb-8693-eee2c93c5985.png)

Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.

Let us see this in action.

Create a new git branch and name it feature/jenkinspipeline-stages

git branch feature/jenkinspipeline-stages

git switch feature/jenkinspipeline-stages

Currently we only have the Build stage. Let us add another stage called Test. Paste the code snippet below and push the new changes to GitHub.

![image](https://user-images.githubusercontent.com/49937302/123514857-8699e280-d6c7-11eb-8d99-b1f2121204e3.png)

To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

Click on the “Administration” button

![image](https://user-images.githubusercontent.com/49937302/123515035-86e6ad80-d6c8-11eb-9bf1-fd3350fe3d16.png)

Navigate to the Ansible project and click on “Scan repository now”

![image](https://user-images.githubusercontent.com/49937302/123515063-aa115d00-d6c8-11eb-8357-4694b2c6b947.png)

Navigate to Scan repository log to check the logs

![image](https://user-images.githubusercontent.com/49937302/123515216-fe1c4180-d6c8-11eb-81ce-a21732700c03.png)

Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.

![image](https://user-images.githubusercontent.com/49937302/123515286-5ce1bb00-d6c9-11eb-91e6-546b8add73f1.png)

In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.

![image](https://user-images.githubusercontent.com/49937302/123515402-d1b4f500-d6c9-11eb-9ed9-74c9e31d0d2c.png)

Create a pull request to merge the latest code into the main branch

After merging the PR, go back into your terminal and switch into the main branch

Pull the latest change

Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an `echo` command like we have in `build` and `test` stages)

   - Package 
   
   - Deploy 
  
   - Clean up

Verify in Blue Ocean that all the stages are working, then merge the feature branch to the main branch

Eventually, the main branch should have a successful pipeline like this in blue ocean

![image](https://user-images.githubusercontent.com/49937302/123631557-76aa0c00-d849-11eb-825c-cdd8f6130549.png)

![image](https://user-images.githubusercontent.com/49937302/123631487-5f6b1e80-d849-11eb-9d9d-e9fdda387369.png)

![image](https://user-images.githubusercontent.com/49937302/123631664-980af800-d849-11eb-992d-15d11abae04a.png)

# Running Ansible Playbook from Jenkins

Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:

Installing Ansible on Jenkins

sudo apt install ansible -y

Installing Ansible plugin in Jenkins UI

![image](https://user-images.githubusercontent.com/49937302/123632574-aefe1a00-d84a-11eb-92eb-5f63d4bb94fb.png)

![image](https://user-images.githubusercontent.com/49937302/123632688-cdfcac00-d84a-11eb-944b-c30d9b593eb2.png)

manage jenkins,global tool configuration

![image](https://user-images.githubusercontent.com/49937302/123633925-634c7000-d84c-11eb-81cc-a9315212fb4a.png)

Create new pipeline



Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)
