FROM httpd:2.4.43-alpine
WORKDIR /usr/local/apache2/htdocs/
COPY index.html .
COPY custom.conf .
RUN cat custom.conf > /usr/local/apache2/conf/httpd.conf