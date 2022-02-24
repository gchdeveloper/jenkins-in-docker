# Run Jenkins in Docker

***This project is intended for my personal use only, it should not be depended on for professional projects, use at your own risk.***

### What It Does

This project allows [Jenkins](https://www.jenkins.io/) to be run with a single docker-compose command without needing to install anything other than [Docker](https://docs.docker.com/) (which you can get [here](https://docs.docker.com/get-docker/)). If your system runs Docker, it should be able to use this to run Jenkins.

### How To Run It

1. Clone this project.
2. Run `docker-compose up -d` to start the containers.
3. Navigate to [http://localhost:8080](http://localhost:8080) to access Jenkins. 
4. Configure Jenkins following the steps [here](https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/#setup-wizard).
5. Stop the containers with `docker-compose down`.
