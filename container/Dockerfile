FROM alpine:3.21.2 as builder

COPY ./detect_arch /opt/detect_arch
COPY ./kubeconfig.yaml /home/kubie/.kube/config

RUN apk add unzip && \
    adduser \
        --disabled-password \
        --gecos "" \
        --uid "1000" \
        --shell bash \
        "kubie"

RUN arch=$(/opt/detect_arch) && \
    # kubectl
    wget "https://dl.k8s.io/release/$(wget -q -O- https://dl.k8s.io/release/stable.txt)/bin/linux/${arch}/kubectl" && \
    install -o kubie -g kubie -m 0755 kubectl /usr/local/bin/kubectl && \
    # helm
    wget "https://get.helm.sh/helm-v3.12.3-linux-${arch}.tar.gz" && \
    tar -zxvf helm-v3.12.3-linux-${arch}.tar.gz && \
    install -o kubie -g kubie -m 0755 linux-${arch}/helm /usr/local/bin/helm

RUN wget https://raw.githubusercontent.com/kubermatic/fubectl/master/fubectl.source -P /home/kubie && \
    echo "export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$'" >> /etc/profile && \
    echo "[ -f /home/kubie/fubectl.source ] && source /home/kubie/fubectl.source" >> /etc/profile && \
    echo "cd /home/kubie" >> /etc/profile
    
FROM alpine:3.21.2

COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /etc/profile /etc/profile 
COPY --chown=kubie:1000 --from=builder /home/kubie /home/kubie

RUN apk add --no-cache tini ttyd bash fzf

USER 1000

ENV TINI_KILL_PROCESS_GROUP=1

ENTRYPOINT ["/sbin/tini", "--"]
CMD [ "ttyd", "-s", "3", "-t", "titleFixed=kube-session", "-t", "rendererType=webgl", "-t", "disableLeaveAlert=true", "/bin/bash", "-i", "-l" ]
