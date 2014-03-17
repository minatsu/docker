FROM centos:latest

RUN yum install sudo \
  passwd \
  openssh \
  openssh-clients \
  openssh-server \
  python-setuptools \
  httpd \
  vim -y

# webmin
RUN wget -O /tmp/webmin-1.670.rpm "http://downloads.sourceforge.net/project/webadmin/webmin/1.670/webmin-1.670-1.noarch.rpm?r=http%3A%2F%2Fwww.webmin.com%2Fdownload.html&ts=1394605927&use_mirror=jaist"
RUN rpm -U /tmp/webmin-1.670.rpm
RUN touch /var/webmin/miniserv.log

# root password
RUN echo 'root:hogehoge' | chpasswd

EXPOSE 22 80 10000

CMD /etc/init.d/webmin restart && wait && tail -f /var/webmin/miniserv.log

