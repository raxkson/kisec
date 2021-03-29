# Dockerfile
FROM ubuntu:18.04
RUN echo "nameserver 8.8.8.8" >> /etc/resolv.conf
RUN echo "nameserver 8.8.4.4" >> /etc/resolv.conf
 
RUN apt update
RUN apt upgrade -y
RUN apt install git -y

RUN apt install apache2 -y
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_PID_FILE  /var/run/apache2/apache2.pid 
 
RUN apt install software-properties-common -y
 
RUN add-apt-repository ppa:ondrej/php
RUN apt update
 
ENV DEBIAN_FRONTEND=noninteractive 
 
RUN apt install php7.3 php7.3-common php7.3-cli -y
RUN apt install php7.3-bcmath php7.3-bz2 php7.3-curl php7.3-gd php7.3-intl php7.3-json php7.3-mbstring php7.3-readline php7.3-xml php7.3-zip -y
RUN apt install libapache2-mod-php7.3

RUN mkdir /home/visual
RUN cd /home/visual && git clone https://github.com/kisecADCTF/test.git
RUN mv /home/visual/test/* /home/visual
RUN cp /home/visual/visual.conf /etc/apache2/sites-avaliable
RUN rm /home/visual/visual.conf 
RUN rm -rf /home/visual/test
RUN mv /home/visual/* /var/www/html/

RUN ln -s /etc/apache2/sites-available/visual.conf /etc/apache2/sites-enabled/
 
EXPOSE 80
 
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
