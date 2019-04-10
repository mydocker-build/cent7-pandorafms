FROM centos:centos6
MAINTAINER Pandora FMS Team <info@pandorafms.com>

RUN { \
	echo '[EPEL]'; \
	echo 'name = CentOS Epel'; \
	echo 'baseurl = http://dl.fedoraproject.org/pub/epel/6/x86_64'; \
	echo 'enabled=1'; \
	echo 'gpgcheck=0'; \
} > /etc/yum.repos.d/extra_repos.repo

RUN { \
        echo '[artica_pandorafms]'; \
        echo 'name=CentOS6 - PandoraFMS official repo'; \
        echo 'baseurl=http://artica.es/centos6'; \
        echo 'gpgcheck=0'; \
        echo 'enabled=1'; \
} > /etc/yum.repos.d/pandorafms.repo

RUN yum -y update; yum clean all;
RUN yum install -y \ 
	git \
	httpd \
	mod_ssl \
	cronie \
	ntp \
	openldap \
	nfdump \
	wget \
	curl \
	openldap \
	plymouth \
	xterm \
	php \ 
	php-gd \ 
	graphviz \ 
	php-mysql \ 
	php-pear-DB \ 
	php-pear \
	php-pdo \
	php-mbstring \ 
	php-ldap \ 
	php-snmp \ 
	php-ldap \ 
	php-common \ 
	php-zip \ 
	nmap \
	net-snmp-utils \
	mod_ssl \
	xprobe2

# Set PHP timezone
RUN sed -i "s/^;date.timezone =$/date.timezone = \"Asia\/Phnom_Penh\"/" /etc/php.ini

# Secure Apache server
## Disable CentOS Welcome Page
RUN sed -i 's/^\([^#]\)/#\1/g' /etc/httpd/conf.d/welcome.conf

## Turn off directory listing, Disable Apache's FollowSymLinks, Turn off server-side includes (SSI) and CGI execution
RUN sed -i 's/^\([^#]*\)Options Indexes FollowSymLinks/\1Options -Indexes +SymLinksifOwnerMatch -ExecCGI -Includes/g' /etc/httpd/conf/httpd.conf

## Hide the Apache version, secure from clickjacking attacks, disable ETag, secure from XSS attacks and protect cookies with HTTPOnly flag
RUN echo $'\n\
ServerSignature Off\n\
ServerTokens Prod\n\
Header append X-FRAME-OPTIONS "SAMEORIGIN"\n\
FileETag None\n\
<IfModule mod_headers.c>\n\
    Header set X-XSS-Protection "1; mode=block"\n\
</IfModule>\n\
# Redirect to HTTPs\n\
<VirtualHost *:*>\n\
    ServerName *\n\
    Redirect permanent / https://*\n\
    Header set Strict-Transport-Security "max-age=31536000"\n\
</VirtualHost>\n'\
>> /etc/httpd/conf/httpd.conf

## Disable unnecessary modules in /etc/httpd/conf.modules.d/00-base.conf
#RUN sed -i '/mod_cache.so/ s/^/#/' /etc/httpd/conf.modules.d/00-base.conf && \
#    sed -i '/mod_cache_disk.so/ s/^/#/' /etc/httpd/conf.modules.d/00-base.conf && \
#    sed -i '/mod_substitute.so/ s/^/#/' /etc/httpd/conf.modules.d/00-base.conf && \
#    sed -i '/mod_userdir.so/ s/^/#/' /etc/httpd/conf.modules.d/00-base.conf
#
## Disable everything in /etc/httpd/conf.modules.d/00-dav.conf, 00-lua.conf, 00-proxy.conf and 01-cgi.conf
#RUN sed -i 's/^/#/g' /etc/httpd/conf.modules.d/00-dav.conf && \
#    sed -i 's/^/#/g' /etc/httpd/conf.modules.d/00-lua.conf && \
#    sed -i 's/^/#/g' /etc/httpd/conf.modules.d/00-proxy.conf && \
#    sed -i 's/^/#/g' /etc/httpd/conf.modules.d/01-cgi.conf

# SSL Settings
RUN mkdir -m 700 -p /etc/httpd/ssl/private && \
    sed -i '/^SSLProtocol/ s/^/#/' /etc/httpd/conf.d/ssl.conf && \
    sed -i '/^SSLCipherSuite/ s/^/#/' /etc/httpd/conf.d/ssl.conf && \
    sed -i "s@^#DocumentRoot .*@DocumentRoot "/var/www/html/pandora_console/"@" /etc/httpd/conf.d/ssl.conf && \
    sed -i "s@^#ServerName .*@ServerName *:443@" /etc/httpd/conf.d/ssl.conf && \
    sed -i "s@^SSLCertificateFile .*@SSLCertificateFile /etc/httpd/ssl/apache-selfsigned.crt@" /etc/httpd/conf.d/ssl.conf && \
    sed -i "s@^SSLCertificateKeyFile .*@SSLCertificateKeyFile /etc/httpd/ssl/private/apache-selfsigned.key@" /etc/httpd/conf.d/ssl.conf
ADD ./ssl/* /etc/httpd/ssl/private/

#Clone the repo
RUN git clone -b develop https://github.com/pandorafms/pandorafms.git /tmp/pandorafms

#Exposing ports for: HTTP, SNMP Traps, Tentacle protocol
EXPOSE 80 162/udp 443 41121

# Simple startup script to avoid some issues observed with container restart
ADD docker_entrypoint.sh /entrypoint.sh
RUN chmod -v +x /entrypoint.sh

CMD ["/entrypoint.sh"]

