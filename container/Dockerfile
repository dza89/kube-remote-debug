FROM alpine:3.16.2

COPY ./detect_arch /opt/detect_arch

RUN apk add --no-cache tini curl ttyd unzip ca-certificates && \
    adduser \
    --disabled-password \
    --gecos "" \
    --no-create-home \
    --uid "1000" \
    "kubie" && \
    export arch=$(/opt/detect_arch) && \
    # ngrok
    wget -q https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-${arch}.zip && \
    unzip ngrok-stable-linux-${arch}.zip && \
    mv ngrok /bin/ngrok && \
    rm ngrok-stable-linux-${arch}.zip && \
    echo "web_addr: 0.0.0.0:4040" > /root/ngrok.yml && \
    echo "export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '" >> /etc/profile && \
    echo "alias poweroff='kill 1'" >> /etc/profile && \
    # kubectl
    wget "https://dl.k8s.io/release/$(wget -q -O- https://dl.k8s.io/release/stable.txt)/bin/linux/${arch}/kubectl" && \
    install -o kubie -g kubie -m 0755 kubectl /usr/local/bin/kubectl && \
    # helm
    wget "https://get.helm.sh/helm-v3.10.0-linux-${arch}.tar.gz" && \
    tar -zxvf helm-v3.10.0-linux-${arch}.tar.gz && \
    install -o kubie -g kubie -m 0755 linux-${arch}/helm /usr/local/bin/helm

ENV TINI_KILL_PROCESS_GROUP=1

USER 1000
ENTRYPOINT ["/sbin/tini", "--"]
CMD [ "ttyd", "-s", "3", "-t", "titleFixed=/bin/sh", "-t", "rendererType=webgl", "-t", "disableLeaveAlert=true", "/bin/sh", "-i", "-l" ]