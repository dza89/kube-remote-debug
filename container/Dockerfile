FROM alpine:3.18.2 as builder

COPY ./detect_arch /opt/detect_arch

RUN apk add --no-cache unzip bash && \
    adduser \
    --disabled-password \
    --gecos "" \
    --uid "1000" \
    --shell bash \
    "kubie" && \
    export arch=$(/opt/detect_arch) && \
    # kubectl
    wget "https://dl.k8s.io/release/$(wget -q -O- https://dl.k8s.io/release/stable.txt)/bin/linux/${arch}/kubectl" && \
    install -o kubie -g kubie -m 0755 kubectl /usr/local/bin/kubectl && \
    # helm
    wget "https://get.helm.sh/helm-v3.10.0-linux-${arch}.tar.gz" && \
    tar -zxvf helm-v3.10.0-linux-${arch}.tar.gz && \
    install -o kubie -g kubie -m 0755 linux-${arch}/helm /usr/local/bin/helm && \
    wget https://raw.githubusercontent.com/kubermatic/fubectl/master/fubectl.source -P /home/kubie && \
    echo "[ -f ~/fubectl.source ] && source ~/fubectl.source" >> /home/kubie/.bashrc


FROM alpine:3.18.2

RUN apk add --no-cache tini ttyd bash fzf && \
    echo "export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '" >> /etc/profile

COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /home/kubie /home/kubie

USER 1000

ENV TINI_KILL_PROCESS_GROUP=1

ENTRYPOINT ["/sbin/tini", "--"]
CMD [ "ttyd", "-s", "3", "-t", "titleFixed=/bin/bash", "-t", "rendererType=webgl", "-t", "disableLeaveAlert=true", "/bin/bash", "-i", "-l" ]
