FROM ubuntu:latest

# Configurer les locales
RUN apt update -y && apt upgrade -y && apt install -y locales && \
    localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

# Installer les outils nécessaires
RUN apt install -y ssh wget unzip curl jq

# Télécharger et installer cloudflared
RUN wget https://github.com/cloudflare/cloudflared/releases/download/2024.9.1/cloudflared-linux-amd64.deb && \
    dpkg -i cloudflared-linux-amd64.deb && \
    rm cloudflared-linux-amd64.deb

# Configurer SSH pour autoriser la connexion root
RUN mkdir /run/sshd && \
    echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config && \
    echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config && \
    echo "root:root" | chpasswd

# Créer le script de démarrage
RUN echo "#!/bin/bash\n\
/usr/sbin/sshd &\n\
cloudflared tunnel --url ssh://localhost:22 --no-autoupdate --metrics 127.0.0.1:42679 &\n\
sleep 5\n\
curl --silent --show-error http://localhost:42679/metrics " > /start.sh

# Rendre le script exécutable
RUN chmod +x /start.sh

# Exposer les ports nécessaires
EXPOSE 22

# Commande par défaut
CMD ["/bin/bash", "/start.sh"]
