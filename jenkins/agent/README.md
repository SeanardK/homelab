Jenkins Agent (Docker Compose)

This folder contains a custom Jenkins inbound agent image and a docker-compose file to run it.

What it provides:
- Base on the official `jenkins/inbound-agent` with added tools: Docker CLI, `docker-compose`, `git`, `maven`, `node`, `npm`, `python3`/`pip`, `kubectl`, `helm`, and `awscli`.

Quick start:

1. Build and start the agent (from this directory):

```bash
docker compose -f docker-compose.agent.yml up -d --build
```

2. Replace the placeholder `JENKINS_SECRET` and `JENKINS_AGENT_NAME` in `docker-compose.agent.yml` with the actual values you get when you create a new JNLP agent on your Jenkins controller.

   - In Jenkins UI: Manage Jenkins → Manage Nodes and Clouds → New Node → Permanent Agent. After creation, use the shown agent secret or the launch command.

3. Ensure `agent_home` directory exists and is owned by UID 1000 (jenkins):

```bash
mkdir -p agent_home
sudo chown -R 1000:1000 agent_home
```

Notes and security:
- The compose mounts `/var/run/docker.sock` so the agent can use the host Docker daemon. This grants the container access to the host Docker engine — treat it as privileged access.
- If you prefer stronger isolation, consider running a Docker-in-Docker (dind) setup or use separate builder VMs.
- You can customize `Dockerfile.agent` to add or remove tools depending on your CI needs.

If you want, I can:
- Add a `plugins.txt` and Dockerfile to preinstall Jenkins plugins in the controller image.
- Add a small example Jenkinsfile that demonstrates using Docker, Maven, and Node in a pipeline.
