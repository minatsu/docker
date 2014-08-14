FROM centos:latest

MAINTAINER minatsu

RUN yum install sudo \
  passwd \
  openssh \
  openssh-clients \
  openssh-server \
  python-setuptools \
  wget \
  httpd \
  vim -y

RUN yum groupinstall -y "Development Tools"

RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/#UsePAM no/UsePAM no/g' /etc/ssh/sshd_config
RUN /etc/init.d/sshd start
RUN /etc/init.d/sshd stop

# root password
RUN echo 'root:hogehoge' | chpasswd

EXPOSE 22 80

CMD /usr/sbin/sshd -D

