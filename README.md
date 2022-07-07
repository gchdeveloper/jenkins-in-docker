# Run Jenkins in Docker

***This project is intended for my personal use only, it should not be depended on for professional projects, use at your own risk.***

### What It Does

This project allows [Jenkins](https://www.jenkins.io/) to be run with a single docker-compose command without needing to install anything other than [Docker](https://docs.docker.com/) (which you can get [here](https://docs.docker.com/get-docker/)) and Git. If your system runs Docker, it should be able to use this to run Jenkins.

### How To Run It

*All instructions tested on an EC2 Amazon Linux 2 instance (t2.large, x86 architecture).*

1. Install git 
   1. Install with `yum install git`
   2. Test installation with `git --version`
2. Install docker 
   1. Install with `yum install docker`
   2. Add current user to `docker` group with `sudo usermod -aG docker $USER`
   3. Log out and log back in to refresh the user's group membership
      1. Verify `docker` group membership with `groups`
   4. Test installation with `docker run hello-world`
   5. See [this page](https://docs.docker.com/engine/install/linux-postinstall/) if there are docker issues post-install.
3. Install docker compose
   1. Install manually with
      1. `DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}`
      2. `mkdir -p $DOCKER_CONFIG/cli-plugins`
      3. `curl -SL https://github.com/docker/compose/releases/download/v2.6.1/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose`
   2. Set exectuable permissions with `chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose`
   3. Test installation with `docker compose version`
   4. See [this page](https://docs.docker.com/compose/install/compose-plugin/#install-the-plugin-manually) if there are docker compose issues post-install.
4. Run Jenkins
   1. Clone this project `git clone https://github.com/gchdeveloper/jenkins-in-docker.git`.
   2. cd into the `jenkins-in-docker` directory
   3. Start the containers with `docker-compose up`
   4. Test installation by navigating to [http://localhost:8080](http://localhost:8080) to access Jenkins

### How To Configure It
1. Configure Jenkins
   1. Navigate to [http://localhost:8080](http://localhost:8080) to access Jenkins.
   2. Unlock Jenkins with the initial admin password 
      1. The password will be prominently displayed in the server log near the end
      2. *or* it can be found in this file on the server: `/var/jenkins_home/secrets/initialAdminPassword`
   3. Create the first admin user by filling in the displayed form
   4. Accept the initial value for the Jenkins URL (this can be changed later by navigating to Manage Jenkins -> Configure System -> Jenkins URL)
   5. Accept the defaults for the plugins to be installed (some will fail, but Jenkins will still run)
2. Create an SSH keypair for Jenkins credentials
   1. Create the key pair
      1. Create with `ssh-keygen -b 4096 -t rsa -C "build-box-jenkins"` (The label with `-C` can be any label)
      2. Save the key as `~/.ssh/jenkins_rsa` (not strictly necessary, can use the default or another path)
      3. Use an empty passphrase (not strictly necessary, but simpler, can configure Jenkins to use key's passphrase)
      4. Add the key to the SSH agent
         1. Start the agent in the background with `eval "$(ssh-agent -s)"` (should display agent pid)
         2. Add the key with `ssh-add ~/.ssh/jenkins_rsa` (should display identity added)
      5. Test the key by connecting to github with `ssh -T git@github.com`m (should display a success message)
      6. See [this page](https://inst.eecs.berkeley.edu/~cs61c/sp15/labs/00/github_ssh_key_guide/sshkeys.html) if there are any errors
   2. Add the key to Jenkins credentials
      1. Navigate to Manage Jenkins -> Manage Credentials, select `global` domain, select Add Credentials
      2. Select:
         1. `Kind`: SSH Username with Private Key
         2. `Scope`: Global
         3. `ID`: build-box-jenkins (or any descriptive name, this will identify the credentials used for the build in the build logs)
         4. `Description`: any or none (will appear when selecting this key from Jenkins credentials lists)
         5. `Username`: jenkins (or any descriptive name, this name will appear in the Jenkins server logs)
         6. `Private Key`: select 'Enter Directly', enter the contents of `~/.ssh/jenkins_rsa`

### How To Test It
1. Test Jenkins by creating a test pipeline for a simple [github project](https://github.com/gchdeveloper/simple-java-maven-app)
   1. Add the Jenkins credentials key as a deploy key for the project
      1. Navigate to [https://github.com/gchdeveloper/simple-java-maven-app](https://github.com/gchdeveloper/simple-java-maven-app), Settings -> Deploy keys, select Add deploy key
      2. Select:
         1. `Title`: build-box-jenkins (or any descriptive title)
         2. `Key`: enter the contents of `~/.ssh/jenkins_rsa.pub` (file contents should start with `ssh-rsa `)
   2. Create the project pipeline
      1. Navigate to Dashboard -> New Item
      2. Enter a descriptive name (`test`), select Pipeline, select OK
      3. Scroll down to Pipeline, select: 
         1. `Definition`: Pipeline script from SCM
         2. `SCM`: Git 
         3. `Repository URL`: git@github.com:gchdeveloper/simple-java-maven-app.git
            1. After a few seconds, should get error message: `Failed to connect to repository:...Permission denied...`
         4. `Credentials`: jenkins
            1. After a few seconds, error message should disappear
      4. Select Save
   3. Run the pipeline
      1. Navigate to Dashboard
      2. Select Schedule a Build (icon: clock with green arrow) for the pipeline in the pipelines table
      3. Watch the build progress
         1. Select Open Blue Ocean for builds UI
         2. Select pipeline from the pipelines table
         3. Select running build from the builds table (status icon should be a progress gif)
         4. Can select build stages, select stage steps, and see step output
   4. See [this page](https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/#create-your-pipeline-project-in-jenkins) if there are pipeline setup errors

### How To Secure It

### Creating Pipelines
6. Configure Jenkins following the steps [here](https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/#setup-wizard).
7. Stop the containers with `docker-compose down`.
