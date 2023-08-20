    FROM alpine:3.18.3 as builder

COPY ./detect_arch /opt/detect_arch

RUN apk add --no-cache unzip bash && \
    adduser \
    --disabled-password \
    --gecos "" \
    --uid "1000" \
    --shell bash \
    "kubie" && \
    arch=$(/opt/detect_arch) && \
    # kubectl
    wget "https://dl.k8s.io/release/$(wget -q -O- https://dl.k8s.io/release/stable.txt)/bin/linux/${arch}/kubectl" && \
    install -o kubie -g kubie -m 0755 kubectl /usr/local/bin/kubectl && \
    # helm
    wget "https://get.helm.sh/helm-v3.12.3-linux-${arch}.tar.gz" && \
    tar -zxvf helm-v3.12.3-linux-${arch}.tar.gz && \
    install -o kubie -g kubie -m 0755 linux-${arch}/helm /usr/local/bin/helm && \
    wget https://raw.githubusercontent.com/kubermatic/fubectl/master/fubectl.source -P /home/kubie
    

FROM alpine:3.18.3

COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /home/kubie /home/kubie

RUN apk add --no-cache tini ttyd bash fzf && \
    echo "export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '" >> /etc/profile && \
    echo "[ -f /home/kubie/fubectl.source ] && source /home/kubie/fubectl.source" >> /etc/profile && \
    chown -R kubie:1000 /home/kubie/

COPY ./kubeconfig.yaml /home/kubie/.kube/config

USER 1000

ENV TINI_KILL_PROCESS_GROUP=1

ENTRYPOINT ["/sbin/tini", "--"]
CMD [ "ttyd", "-s", "3", "-t", "titleFixed=kube-session", "-t", "rendererType=webgl", "-t", "disableLeaveAlert=true", "/bin/bash", "-i", "-l" ]
