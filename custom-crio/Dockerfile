FROM quay.io/fedora/fedora:37 as builder
ENV CRIO_REPO https://github.com/vrutkovs/cri-o
ENV CRIO_REPO_COMMIT 7713f00fd
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

FROM registry.ci.openshift.org/origin/4.12:machine-os-content
COPY --from=builder /go/src/github.com/cri-o/cri-o/bin/crio /usr/bin/crio
ENV container oci
RUN ostree container commit
