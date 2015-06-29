FROM ubuntu:15.04

MAINTAINER Christopher Annanie

RUN apt-get update && apt-get install -y python-thrift
COPY python /examples
WORKDIR /examples
CMD python client.py
