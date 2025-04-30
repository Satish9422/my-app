# Use the official Jenkins LTS image
FROM jenkins/jenkins:lts

USER root

# Install kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    chmod +x kubectl && \
    mv kubectl /usr/local/bin/

# Optional: verify installation
RUN kubectl version --client

# Return to Jenkins user
USER jenkins
