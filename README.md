This workspace is originally from linkedin learning course https://www.linkedin.com/learning/docker-continuous-delivery. The instructor has used AWS for Jenkins and AWS ECR for repository. 

The Jenkinsfile in this version is designed for the setup below only:-
 a. Dockers for Windows
 b. Docker Hub as registry
 c. Jenkins Docker Image - jenkins/jenkins/ latest

The jenkins image i build has fixed a lot of issues to enable jenkins running in Docker for Windows to build docker images. You can get them here - https://hub.docker.com/repository/docker/wctan/jenkins_with_docker_cli

# example-webapp

generated using Luminus version "2.9.12.25"

FIXME

## Prerequisites

You will need [Leiningen][1] 2.0 or above installed.

[1]: https://github.com/technomancy/leiningen

## Running

To start a web server for the application, run:

    lein run 

## License

Copyright © 2018 FIXME
