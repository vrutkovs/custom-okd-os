FROM quay.io/fedora/fedora:37 as crio-builder
ENV CRIO_REPO https://github.com/vrutkovs/cri-o
ENV CRIO_REPO_COMMIT 939ed3a00
RUN dnf install -y \
  git-core \
  containers-common \
  device-mapper-devel \
  git \
  glib2-devel \
  glibc-devel \
  glibc-static \
  go \
  gpgme-devel \
  libassuan-devel \
  libgpg-error-devel \
  libseccomp-devel \
  libselinux-devel \
  pkgconfig \
  make \
  runc \
    && dnf clean all
WORKDIR /go/src/github.com/cri-o/cri-o
RUN git init \
    && git remote add origin ${CRIO_REPO} \
    && git fetch origin \
    && git reset ${CRIO_REPO_COMMIT} --hard
RUN make

FROM quay.io/fedora/fedora:37 as kubelet-builder
ENV KUBERNETES_REPO https://github.com/vrutkovs/kubernetes
ENV KUBERNETES_REPO_COMMIT a45de03dca2
RUN dnf install -y git-core make gcc golang rsync \
    && dnf clean all
WORKDIR /go/src/k8s.io/kubernetes
RUN git init \
    && git remote add origin ${KUBERNETES_REPO} \
    && git fetch origin \
    && git reset ${KUBERNETES_REPO_COMMIT} --hard
RUN make WHAT='cmd/kubelet'

FROM registry.ci.openshift.org/origin/4.12:machine-os-content
COPY --from=crio-builder /go/src/github.com/cri-o/cri-o/bin/crio /usr/bin/crio
COPY --from=kubelet-builder /go/src/k8s.io/kubernetes/_output/local/bin/linux/amd64/kubelet /usr/bin/kubelet
ENV container oci
RUN ostree container commit
