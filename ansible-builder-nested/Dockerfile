FROM registry.redhat.io/ansible-automation-platform-21/ansible-builder-rhel8:1.0

RUN microdnf install -y fuse-overlayfs buildah && \
    microdnf clean all && \
    cp /etc/containers/storage.conf /etc/containers/storage.conf.orig && \
    sed -i 's/driver = "overlay"/driver = "vfs"/' /etc/containers/storage.conf && \
    cp /usr/share/containers/containers.conf /etc/containers/ && \
    sed -i 's/# cgroup_manager = "systemd"/cgroup_manager = "cgroupfs"/' /etc/containers/containers.conf && \
    groupadd -r builder -g 1001 && \
    useradd -u 1001 -r -s /sbin/nologin -g 1001 -g root -G 1001 -m -d /home/builder -c "Builder user" builder  && \
    echo -e "1001:1:999\n1001:1002:64535" > /etc/subuid && \
    echo -e "1001:1:999\n1001:1002:64535" > /etc/subgid && \
    mkdir -p ${HOME}/.config/containers && \
    echo "[storage]" > ${HOME}/.config/containers/storage.conf && \
    echo "driver=\"vfs\"" >> ${HOME}/.config/containers/storage.conf && \
    mkdir -p /home/builder/.local/share/containers && \
    chmod -Rf g+rwx /etc/subuid /etc/subgid /etc/passwd /home/builder && \
    chown -Rf 1001:0 /etc/subuid /etc/subgid /etc/passwd /home/builder

VOLUME /var/lib/containers
VOLUME /home/builder/.local/share/containers

ENV BUILDAH_ISOLATION=chroot

USER 1001

WORKDIR /home/builder
