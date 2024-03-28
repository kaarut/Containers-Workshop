# Extra Facts
## Containers and Docker Facts

- Containers are about portability and resource utilization.
- Containers don’t exist as a first-class object - Linux namespaces and cgroups work together to create containers.
- Multiple processes can run in the same “container”, this only means the processes share the same namespaces and cgroup.
- Docker is just one tool of many which you can use to work with containers.
- Docker works three main jobs: packaging apps into images, distributing images and running containers from images.
- Container orchestration, security and building good images take effort and experience. They are complex topics by themselves.
