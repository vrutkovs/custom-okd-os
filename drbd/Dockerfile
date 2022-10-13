FROM quay.io/fedora/fedora:37 as builder
ENV DRBD_VERSION 9.1.11
ENV KERNEL_VERSION 5.17.0
ENV KERNEL_VERSION_RELEASE 128.fc37
WORKDIR /build/
RUN dnf install -y gcc make patch kmod cpio coccinelle diffutils binutils \
        https://kojipkgs.fedoraproject.org//packages/kernel/${KERNEL_VERSION}/${KERNEL_VERSION_RELEASE}/x86_64/kernel-${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64.rpm \
        https://kojipkgs.fedoraproject.org//packages/kernel/${KERNEL_VERSION}/${KERNEL_VERSION_RELEASE}/x86_64/kernel-core-${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64.rpm \
        https://kojipkgs.fedoraproject.org//packages/kernel/${KERNEL_VERSION}/${KERNEL_VERSION_RELEASE}/x86_64/kernel-modules-${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64.rpm \
        https://kojipkgs.fedoraproject.org//packages/kernel/${KERNEL_VERSION}/${KERNEL_VERSION_RELEASE}/x86_64/kernel-devel-${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64.rpm
RUN curl -fsSL https://pkg.linbit.com/downloads/drbd/9/drbd-${DRBD_VERSION}.tar.gz | tar -xz \
    && cd drbd-${DRBD_VERSION} \
    && make KDIR=/usr/src/kernels/${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64

FROM registry.ci.openshift.org/origin/4.12:machine-os-content
ENV KERNEL_VERSION 5.17.0
ENV KERNEL_VERSION_RELEASE 128.fc37
RUN rpm-ostree override replace \
        https://kojipkgs.fedoraproject.org//packages/kernel/${KERNEL_VERSION}/${KERNEL_VERSION_RELEASE}/x86_64/kernel-${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64.rpm \
        https://kojipkgs.fedoraproject.org//packages/kernel/${KERNEL_VERSION}/${KERNEL_VERSION_RELEASE}/x86_64/kernel-core-${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64.rpm \
        https://kojipkgs.fedoraproject.org//packages/kernel/${KERNEL_VERSION}/${KERNEL_VERSION_RELEASE}/x86_64/kernel-modules-${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64.rpm \
    && rpm-ostree cleanup -m
COPY --from=builder /build/drbd-9.1.11/drbd/*.ko /lib/modules/${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64
RUN rm -rf /lib/modules/${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64/kernel/drivers/block/drbd/drbd.ko.xz \
    && depmod -a -F /lib/modules/${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64/System.map ${KERNEL_VERSION}-${KERNEL_VERSION_RELEASE}.x86_64 \
    && ostree container commit
