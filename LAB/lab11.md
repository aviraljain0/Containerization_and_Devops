# Experiment 11: Container Orchestration with Docker Stack

## Objective

To understand and implement container orchestration using Docker Stack and Docker Swarm, enabling deployment and management of multi-container applications across a cluster of Docker nodes.

---

## Theory

### Docker Stack

**Docker Stack** is a collection of interrelated services that share dependencies and can be orchestrated and scaled together. It uses a `docker-compose.yml` (version 3+) file to define the stack configuration and deploys it on a **Docker Swarm**.

### Docker Swarm

**Docker Swarm** is Docker's native clustering and orchestration tool. It turns a pool of Docker hosts into a single virtual host. Key components:

- **Manager Node**: Manages cluster state, scheduling, and orchestration.
- **Worker Node**: Executes the tasks assigned by the manager.
- **Service**: The definition of tasks to execute in the swarm.
- **Task**: A running container which is part of a swarm service.

### Key Concepts

| Concept | Description |
|--------|-------------|
| Stack | A group of interrelated services deployed together |
| Service | A definition of how containers should run (replicas, image, ports) |
| Replica | Individual container instances of a service |
| Overlay Network | Virtual network allowing communication between containers across nodes |
| Volume | Persistent storage for stateful services |

---

## Prerequisites

- Docker Engine installed (version 1.13+)
- Docker Compose file format version 3 or higher
- Basic knowledge of Docker and Docker Compose
- Terminal/Command Prompt access

---

## Environment Setup

### Check Docker Version

```bash
docker --version
docker-compose --version
```

### Initialize Docker Swarm

```bash
# Initialize swarm on the manager node
docker swarm init

# Output will show a token to join worker nodes:
# docker swarm join --token <TOKEN> <MANAGER-IP>:2377
```

### Verify Swarm Status

```bash
docker info | grep -i swarm
docker node ls
```

---

## Experiment Steps

### Step 0: Docker Compose File (`docker-compose.yml`)

The following stack file defines the WordPress and MySQL services used in this experiment:

![docker-compose.yml file](../Screenshots/Lab11_s/lab11.1.png)

---

### Step 1: Stop Running Containers

Before initializing Swarm, ensure no previous containers are running using `docker compose down` followed by `docker ps` to confirm a clean state.

```bash
docker compose down
docker ps
```

![docker compose down and docker ps output](../Screenshots/Lab11_s/lab11.2.png)

---

### Step 2: Initialize Docker Swarm

Initialize the current machine as a Swarm manager node using `docker swarm init`.

```bash
docker swarm init
```

![docker swarm init output](../Screenshots/Lab11_s/lab11.3.png)

---

### Step 3: Verify Swarm Nodes

List all nodes in the Swarm to confirm the manager node is active.

```bash
docker node ls
```

![docker node ls output](../Screenshots/Lab11_s/lab11.4.png)

---

### Step 4: Deploy the Stack

Deploy the WordPress stack using the `docker-compose.yml` file with the stack name `wpstack`.

```bash
docker stack deploy -c docker-compose.yml wpstack
```

![docker stack deploy output](../Screenshots/Lab11_s/lab11.5.png)

---

### Step 5: List All Services

Verify that all services in the stack are running correctly.

```bash
docker service ls
```

![docker service ls output](../Screenshots/Lab11_s/lab11.6.png)

---

### Step 6: Inspect WordPress Service Tasks

View the tasks (containers) running for the `wpstack_wordpress` service.

```bash
docker service ps wpstack_wordpress
```

![docker service ps wpstack_wordpress output](../Screenshots/Lab11_s/lab11.7.png)

---

### Step 7: Check Running Containers

List all running containers to confirm the WordPress and MySQL containers are active.

```bash
docker ps
```

![docker ps output](../Screenshots/Lab11_s/lab11.8.png)

---

### Step 8: Scale the WordPress Service

Scale the WordPress service to 3 replicas to demonstrate horizontal scaling.

```bash
docker service scale wpstack_wordpress=3
```

![docker service scale output](../Screenshots/Lab11_s/lab11.9.png)

---



### Step 9: Filter WordPress Containers

Use `grep` to filter and display only the running WordPress containers.

```bash
docker ps | grep wordpress
```

![docker ps grep wordpress output](../Screenshots/Lab11_s/lab11.11.png)

---

### Step 10: Remove the Stack

Clean up by removing the entire `wpstack` stack.

```bash
docker stack rm wpstack
```

![docker stack rm wpstack output](../Screenshots/Lab11_s/lab11.10.png)

---

## Observations

Fill in the table after completing the experiment:

| Parameter | Observed Value |
|-----------|---------------|
| Stack name | wpstack |
| Number of services deployed | |
| Initial replicas for `wpstack_wordpress` | |
| Replicas after scaling | 3 |
| Port for WordPress | |
| MySQL service name | |
| Overlay network name | |

---

## Expected Output

1. `docker stack ls` should list `mystack` with 3 services.
2. `docker stack services mystack` should show all services running with the correct replica count.
3. Visiting `http://localhost:5000` should display the visit counter.
4. Visiting `http://localhost:8080` should show the Docker Swarm visualizer with service distribution.

---

## Docker Stack vs Docker Compose

| Feature | Docker Compose | Docker Stack |
|--------|---------------|-------------|
| Scope | Single host | Multi-node cluster (Swarm) |
| Swarm required | No | Yes |
| Scaling | Manual (`--scale`) | Declarative (`deploy.replicas`) |
| Rolling updates | No | Yes |
| Load balancing | No | Built-in |
| Healthchecks | Basic | Advanced with policies |
| Use case | Development | Production |

---

## Common Docker Stack Commands

```bash
# Deploy or update a stack
docker stack deploy -c <compose-file> <stack-name>

# List stacks
docker stack ls

# List services in a stack
docker stack services <stack-name>

# List tasks in a stack
docker stack ps <stack-name>

# Remove a stack
docker stack rm <stack-name>

# Scale a service
docker service scale <service-name>=<replicas>

# Update service image
docker service update --image <new-image> <service-name>

# View service logs
docker service logs <service-name>
```

---

## Troubleshooting

| Problem | Possible Cause | Solution |
|--------|---------------|----------|
| Service not starting | Image not found | Build image before deploying |
| Swarm not initialized | `docker swarm init` not run | Initialize swarm first |
| Port conflict | Port already in use | Change port mapping in compose file |
| Replicas stuck in pending | Resource constraints | Check node availability with `docker node ls` |
| No swarm node available | Not in swarm mode | Run `docker swarm init` |

---

## Viva Questions

1. What is the difference between Docker Compose and Docker Stack?
2. What is a Docker Swarm and what are its key components?
3. What is the purpose of an overlay network in Docker Swarm?
4. How do you perform a rolling update in Docker Stack?
5. What is the significance of `deploy` key in the Compose file version 3?
6. How does Docker Swarm provide load balancing?
7. What are placement constraints and why are they used?
8. How do you monitor and scale services in Docker Stack?
9. What happens to running containers when you run `docker stack rm`?
10. What is the difference between a Service and a Task in Docker Swarm?

---

## Conclusion

In this experiment, we successfully:
- Initialized a Docker Swarm cluster
- Defined a multi-service application using Docker Compose (v3) stack file
- Deployed the application as a Docker Stack
- Scaled services up and down
- Monitored stack status and service logs
- Performed rolling updates
- Cleaned up the deployed stack

Docker Stack with Swarm provides a powerful, production-ready orchestration mechanism with features like rolling updates, load balancing, service scaling, and fault tolerance — all managed declaratively through a single Compose file.

---

## References

- [Docker Stack Documentation](https://docs.docker.com/engine/reference/commandline/stack/)
- [Docker Swarm Overview](https://docs.docker.com/engine/swarm/)
- [Compose File Reference (v3)](https://docs.docker.com/compose/compose-file/compose-file-v3/)
- [Docker Services](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)
