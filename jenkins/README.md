Jenkins Docker Compose

Quick notes:

- Create persistent volume directory:

  mkdir -p jenkins_home
  sudo chown -R 1000:1000 jenkins_home

- Start Jenkins:

  docker compose up -d

- Initial admin password:

  cat jenkins_home/secrets/initialAdminPassword

- If you map `/var/run/docker.sock` the Jenkins container can run Docker-in-Docker builds; be aware this grants the container access to the host Docker daemon.
