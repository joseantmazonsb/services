FROM ubuntu:20.04
RUN apt-get -qq update && apt-get -qq install -qq apt-transport-https gnupg
RUN echo "deb https://apt.wdkro.de/ stable main" > /etc/apt/sources.list.d/easywall.list
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys D88D7689C1624EE3
RUN apt-get -qq update && apt-get -qq install easywall

EXPOSE 12227