FROM jenkins/jenkins:2.228

LABEL maintainer=”RaviKumar”

USER root

RUN mkdir /var/log/jenkins
RUN mkdir /var/cache/jenkins
RUN chown -R  jenkins:jenkins /var/log/jenkins
RUN chown -R jenkins:jenkins /var/cache/jenkins


USER jenkins

#ENV JAVA_OPTS="-Xmx8192m"
#ENV JENKINS_OPTS=" --handlerCountMax=300"
#ENV JENKINS_OPTS="--handlerCountMax=300 --logfile=/var/log/jenkins/jenkins.log"
ENV JENKINS_OPTS="--handlerCountMax=300 --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war"

