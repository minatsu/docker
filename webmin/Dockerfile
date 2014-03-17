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
RUN wget -O /tmp/webmin-1.680.rpm "http://downloads.sourceforge.net/project/webadmin/webmin/1.680/webmin-1.680-1.noarch.rpm?r=&ts=1395022775&use_mirror=jaist"
RUN rpm -U /tmp/webmin-1.680.rpm
RUN touch /var/webmin/miniserv.log

# root password
RUN echo 'root:hogehoge' | chpasswd

EXPOSE 22 80 10000

CMD /etc/init.d/webmin restart && wait && tail -f /var/webmin/miniserv.log

