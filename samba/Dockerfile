
FROM ubuntu:devel
COPY init /.
RUN  apt-get update && \
    mkdir -m777 /data && apt install -y samba && chmod +x init && \
    useradd smb && echo "smb:smb" | chpasswd &&    (echo "smb"; echo "smb") | smbpasswd -a -s smb
ENTRYPOINT ["/bin/bash", "/init"]

