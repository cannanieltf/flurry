FROM ubuntu:15.04
MAINTAINER Christopher Annanie

RUN apt-get update
RUN apt-get --assume-yes install openjdk-8-jdk
RUN apt-get --assume-yes install git
RUN apt-get --assume-yes install unzip
WORKDIR /
RUN git clone https://github.com/cannanieltf/flurry.git
WORKDIR /flurry
RUN ./gradlew build
RUN ./gradlew flurryZip
RUN unzip /flurry/build/distributions/*.zip -d /

#Can I get rid of this hardcoded value?
WORKDIR /flurry-0.2.0-beta/
ENV FLURRY_HOME=/flurry-0.2.0-beta/
COPY application.conf application.conf
COPY startFlurry startFlurry
CMD ./startFlurry
EXPOSE 9090

