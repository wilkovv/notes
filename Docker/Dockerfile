# Build image on ubuntu 22.04 as base.
FROM ubuntu:22.04

# Update ubuntu repos and install wget and java.
RUN ["/bin/bash", "-c","apt update && apt install -y wget openjdk-17-jre-headless"]
RUN ["/bin/bash", "-c","mkdir -p /srv/minecraft/data"]

WORKDIR /srv/minecraft

# Download official 1.19.2 server jar from mojang servers and setup unprivileged minecraft user.
RUN ["wget", "https://piston-data.mojang.com/v1/objects/f69c284232d7c7580bd89a5a4931c3581eae1378/server.jar"]
RUN ["/bin/bash", "-c","useradd -u 1000 minecraft"]
RUN ["/bin/bash", "-c","chown -R minecraft /srv/minecraft"]

# /srv/minecraft/data is mount point for volume with all the server files from host machine
# if no volume is attached all server files will be deleted when container is stopped.
WORKDIR /srv/minecraft/data

# Accept EULA and run server.jar as minecraft user.
USER 1000
ENV MEM=1G
RUN ["/bin/bash", "-c", "echo 'eula=true' > eula.txt"]

# Redirecting stdin with cat - makes it possible to detach and reattach to minecraft console
# with 'docker attach', however container must be run with '-it' options.
CMD ["/bin/bash","-c","cat | java -Xmx$MEM -Xms$MEM -jar /srv/minecraft/server.jar nogui"]