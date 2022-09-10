# Build image on ubuntu 22.04 as base.
FROM ubuntu:22.04

# Update ubuntu repos and install openjdk.
RUN ["/bin/bash", "-c","apt update && apt install -y openjdk-17-jre-headless"]

# Setup new unprivileged minecraft user with /srv/minecraft directory
RUN ["/bin/bash", "-c","useradd -u 1000 minecraft"]
RUN ["/bin/bash", "-c","mkdir -p /srv/minecraft/data && chown -R minecraft /srv/minecraft"]

# Download official 1.19.2 server jar from mojang servers
ADD --chown=1000 ["https://piston-data.mojang.com/v1/objects/f69c284232d7c7580bd89a5a4931c3581eae1378/server.jar","/srv/minecraft/server.jar"]

# /srv/minecraft/data is mount point for volume with all the server files from host machine
# if no volume is attached all server files will be deleted when container is stopped.
WORKDIR /srv/minecraft/data

# Switch to minecraft user
USER 1000

# By default use 1G of RAM
ENV MEM=1G

# Accept EULA
RUN ["/bin/bash", "-c", "echo 'eula=true' > eula.txt"]

# Start server.jar, redirecting stdin with cat - makes it possible to detach and reattach to 
# minecraft console with 'docker attach', however container must be run with '-it' options.
EXPOSE 25565
CMD ["/bin/bash","-c","cat | java -Xmx$MEM -Xms$MEM -jar /srv/minecraft/server.jar nogui"]