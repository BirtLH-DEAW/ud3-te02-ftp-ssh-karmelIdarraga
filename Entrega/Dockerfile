# Utilizamos una imagen oficial de Ubuntu
# FROM ubuntu:18.04
FROM ubuntu:latest

# Damos información sobre la imagen que estamos creando
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + virtual host + protpd + openssl + ssh + git" \
    maintainer="kidarraga <kidarraga@birt.eus>"

# Actualizamos la lista de paquetes e instalamos nano y apache2
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y nano apache2 proftpd openssl proftpd-mod-crypto ssh git\
    && groupadd sitioprimero \
    && useradd kidarraga1 --password "$(openssl passwd -1 "deaw")" -g sitioprimero --shell /bin/false \
    && useradd kidarraga2 -m -d /var/www/html/sitiosegundo --password "$(openssl passwd -1 "deaw")" --shell /bin/bash \
    && chmod 755 -R /var/www/html/sitiosegundo/ \     
    && openssl genpkey -algorithm RSA  -out /etc/ssl/private/kidarraga-ftpserver.key -pkeyopt rsa_keygen_bits:4096 \
    && openssl req -new -key /etc/ssl/private/kidarraga-ftpserver.key -x509 -days 365 -out /etc/ssl/certs/kidarraga-ftpserver.crt  -subj "/C=EU/ST=Bizkaia/L=Mungia/O=Karmel Idarraga/CN=www.kidarraga-birt.eus/emailAddress=kidarraga@birt.eus" \
    && rm -rf /var/lib/apt/lists/*    
    

# Creamos directorios para los sitios web y configuraciones
RUN mkdir -p /var/www/html/sitioprimero /var/www/html/sitiosegundo \
    && chmod 775 -R /var/www/html/sitioprimero/ \
    && chgrp sitioprimero /var/www/html/sitioprimero/

# Copiamos archivos al contenedor
COPY indexPrimero.html indexSegundo.html sitioPrimero.conf sitioSegundo.conf sitioPrimero.key SitioPrimero.cer sshd_config id_rsa /
COPY proftpd /etc/proftpd/

# Movemos los archivos a sus ubicaciones adecuadas
RUN mv /indexPrimero.html /var/www/html/sitioprimero/index.html && \
    mv /indexSegundo.html /var/www/html/sitiosegundo/index.html && \
    mv /sitioPrimero.conf /etc/apache2/sites-available/sitioprimero.conf && \
    mv /sitioSegundo.conf /etc/apache2/sites-available/sitiosegundo.conf && \
    mv /sitioPrimero.key /etc/ssl/private/sitioprimero.key && \
    mv /SitioPrimero.cer /etc/ssl/certs/sitioprimero.cer && \
    mv /sshd_config /etc/ssh/sshd_config && \
    mv id_rsa /etc/id_rsa

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitioprimero.conf && \
    a2ensite sitiosegundo.conf && \
    a2enmod ssl

#Realizamos la configuración necesaria para el GIT
RUN eval "$(ssh-agent -s)" \
    && chmod 700 /etc/id_rsa \
    && ssh-add /etc/id_rsa \
    && ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
    && git clone git@github.com:deaw-birt/UD3-ftp_anonimo.git /srv/ftp/repositorio
#    && git clone git@github.com:eglarrea/Larrea_Manzano_Egoitz_DEAW03_02.git /srv/ftp/repositorio

# Exponemos los puertos
EXPOSE 80
EXPOSE 443
EXPOSE 21
EXPOSE 1022
EXPOSE 50000-50030

# Comando por defecto al iniciar el contenedor que me da error
# CMD ["apache2-foreground"]
# Comando por defecto al iniciar el contenedor omentado en el foro
CMD service proftpd start && \
    service ssh start && \
    apachectl -D FOREGROUND
