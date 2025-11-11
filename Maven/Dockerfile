# Base image pulled from Artifactory (Alpine Linux 3.20)
FROM artifactory.domain.com:443/domain-docker-remote/alpine:3.20

# Set environment variables

# Prevents interactive prompts during package installs
ENV DEBIAN_FRONTEND nointeractive   
 # Defines Jenkins user's home directory
ENV HOME /home/jenkins             

# Create jenkins group (GID 1000) and jenkins user (UID 1000) with home directory
RUN addgroup -g 1000 jenkins && \
    adduser -u 1000 -G jenkins -h /home/jenkins -D jenkins

# Copy custom CA certificates into container
COPY ca-certificates.crt /tmp/ca-certificates
# Append custom certificates to system CA bundle (internal enterprise CA certs for your company’s Artifactory, internal HTTPS endpoints, etc.).
RUN cat /tmp/certificates.crt >> /etc/ssl/certs/ca-certificates.crt

# Add Artifactory public key to Alpine package manager keys
COPY artifactory.domain.com.rsa.pub /etc/apk/keys/
RUN chmod 644 /etc/apk/keys/artifactory.domain.com.rsa.pub

# Configure Alpine repositories to point to Artifactory (with credentials)
RUN echo "https://<artifactory-username>:<artifactory-token>@artifactory.domain.com/artifactory/devsecops-alpine-virtual/v3.20/main" > /etc/apk/repositories &&\
 echo "https://<artifactory-username>:<artifactory-token>@artifactory.domain.com/artifactory/devsecops-alpine-virtual/v3.20/community" >> /etc/apk/repositories &&\
 echo "https://<artifactory-username>:<artifactory-token>@artifactory.domain.com/artifactory/devsecops-alpine-virtual/v3.20/testing" >> /etc/apk/repositories

# Update package index and install required tools (Java, Git, Maven dependencies, etc.)
RUN apk update --allow-untrusted && \
    apk add wget git openjdk21 sshpass curl zip openssh-client tar bash --allow-untrusted

# Copy CA certificates again (redundant step, might be cleanup candidate)
COPY ca-certificates.crt /tmp/ca-certificates
RUN cat /tmp/certificates.crt >> /etc/ssl/certs/ca-certificates.crt

# Set Maven environment variables
ENV PATH "$PATH:/usr/local/maven/bin"
ENV MAVEN_VERSION 3.9.8

# Download Maven from Artifactory, extract, install, and set ownership to jenkins user  
# https://archive.apache.org/dist/maven/ should be configured to ----> devsecops-maven-generic-remote
RUN wget -q https://<artifactory-username>:<artifactory-token>@artifactory.domain.com/artifactory/devsecops-maven-generic-remote/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz && \
    mkdir -p /usr/local/maven && \
    tar -xf apache-maven-${MAVEN_VERSION}-bin.tar.gz && \
    mv apache-maven-${MAVEN_VERSION}/* /usr/local/maven/ && \
    rm -rf apache-maven-${MAVEN_VERSION} && \
    rm -rf apache-maven-${MAVEN_VERSION}-bin.tar.gz && \
    chown -R jenkins:jenkins /usr/local/maven

# Copy custom Java truststore (cacerts) into OpenJDK security directory
COPY cacerts /usr/lib/jvm/java-21-openjdk/lib/security/cacerts
# Adjust ownership and permissions for Jenkins user
RUN chown -R jenkins:jenkins /usr/lib/jvm/java-21-openjdk && \
    chmod 777 /usr/lib/jvm/java-21-openjdk/lib/security/cacerts

# Set working directory to Jenkins home
WORKDIR $HOME

# Define a healthcheck 
HEALTHCHECK CMD java -version || exit 1

# Switch to Jenkins user (non-root execution for security)
USER jenkins
