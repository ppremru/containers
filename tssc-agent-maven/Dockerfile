##############################################
# Stage 1 : Build go-init
##############################################
FROM openshift/origin-release:golang-1.12 AS go-init-builder
WORKDIR  /go/src/github.com/openshift/jenkins
COPY . .
WORKDIR  /go/src/github.com/openshift/jenkins/go-init
RUN go build . && cp go-init /usr/bin

##############################################
# Stage 1.5 : Retrieive oc cli
##############################################
FROM quay.io/openshift/origin-cli as origin-cli

##############################################
# Stage 2 : Build slave-base with go-init
##############################################
#FROM quay.io/openshift/origin-cli
FROM registry.access.redhat.com/ubi8:latest
MAINTAINER OpenShift Developer Services <openshift-dev-services+jenkins@redhat.com>
COPY --from=go-init-builder /usr/bin/go-init /usr/bin/go-init
COPY --from=origin-cli /usr/bin/oc /usr/bin/oc
COPY --from=origin-cli /usr/bin/kubectl /usr/bin/kubectl

# Labels consumed by Red Hat build service
LABEL com.redhat.component="jenkins-slave-base-rhel8-container" \
      name="openshift4/jenkins-slave-base-rhel7" \
      architecture="x86_64" \
      io.k8s.display-name="Jenkins Slave Base" \
      io.k8s.description="The jenkins slave base image is intended to be built on top of, to add your own tools that your jenkins job needs. The slave base image includes all the jenkins logic to operate as a slave, so users just have to dnf install any additional packages their specific jenkins job will need" \
      io.openshift.tags="openshift,jenkins,slave" \
      maintainer="openshift-dev-services+jenkins@redhat.com"      


ENV HOME=/home/jenkins \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

USER root
# Install headless Java
# Install maven for the purpose of testing this out ... this will be a layered Dockerfile as we iterate
RUN curl -L https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 -o /usr/bin/jq && \
    chmod +x /usr/bin/jq && \
    INSTALL_PKGS="bc gettext git java-11-openjdk-headless java-1.8.0-openjdk-headless maven lsof rsync tar unzip which zip bzip2 python36 python3-pip python3-pip-wheel nss_wrapper python3-setuptools python36-devel" && \
    dnf install -y --setopt=tsflags=nodocs --disableplugin=subscription-manager $INSTALL_PKGS && \
    alternatives --set python /usr/bin/python3 && \
    rpm -V  $INSTALL_PKGS && \
    dnf clean all && \
    mkdir -p /home/jenkins && \
    chown -R 1001:0 /home/jenkins && \
    chmod -R g+w /home/jenkins && \
    chmod -R 775 /etc/alternatives && \
    chmod -R 775 /var/lib/alternatives && \
    chmod -R 775 /usr/lib/jvm && \
    chmod 775 /usr/bin && \
    chmod 775 /usr/share/man/man1 && \
    mkdir -p /var/lib/origin && \
    chmod 775 /var/lib/origin && \    
    unlink /usr/bin/java && \
    unlink /usr/bin/jjs && \
    unlink /usr/bin/keytool && \
    unlink /usr/bin/pack200 && \
    unlink /usr/bin/rmid && \
    unlink /usr/bin/rmiregistry && \
    unlink /usr/bin/unpack200 && \
    unlink /usr/share/man/man1/java.1.gz && \
    unlink /usr/share/man/man1/jjs.1.gz && \
    unlink /usr/share/man/man1/keytool.1.gz && \
    unlink /usr/share/man/man1/pack200.1.gz && \
    unlink /usr/share/man/man1/rmid.1.gz && \
    unlink /usr/share/man/man1/rmiregistry.1.gz && \
    unlink /usr/share/man/man1/unpack200.1.gz

# Copy the entrypoint
ADD contrib/bin/* /usr/local/bin/

# Run the Jenkins JNLP client
ENTRYPOINT ["/usr/bin/go-init", "-main", "/usr/local/bin/run-jnlp-client"]

##############################################
# End
##############################################
