FROM rockylinux:8
COPY init /.
RUN dnf install -y samba samba-common samba-client && \
   mkdir -p /srv/ && chmod -R 755 /srv/ && \
   chown -R  nobody:nobody /srv/ && \
   mv /etc/samba/smb.conf /etc/samba/smb.conf.bak

ENTRYPOINT ["/bin/bash", "/init"]

