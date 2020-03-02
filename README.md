## About

This repository provides docker-compose file with custom Dockerfile files to create jenkins master/slave (swarm) with preinstalled docker-cli and azure-cli in slaves.

## Troubleshoots

Jenkins master cannot start due to writing file to /var/jenkins_home permission

```
chown ${USER}:${USER} ./docker/jenkins/master/home
chmod 755 ./docker/jenkins/master/home
```