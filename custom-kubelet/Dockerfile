FROM quay.io/fedora/fedora:37 as artifacts
ENV KUBERNETES_REPO https://github.com/openshift/kubernetes
ENV KUBERNETES_REPO_COMMIT 3ef6ef3cfbd
RUN dnf install -y git-core make gcc golang rsync \
    && dnf clean all
WORKDIR /go/src/k8s.io/kubernetes
RUN git init \
    && git remote add origin ${KUBERNETES_REPO} \
    && git fetch origin --depth=1 \
    && git reset ${KUBERNETES_REPO_COMMIT} --hard
RUN make WHAT='cmd/kubelet'

FROM registry.ci.openshift.org/origin/4.12:machine-os-content
COPY --from=artifacts /go/src/k8s.io/kubernetes/_output/local/bin/linux/amd64/kubelet /usr/bin/kubelet
RUN ostree container commit
