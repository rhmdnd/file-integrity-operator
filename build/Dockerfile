# Step one: build file-integrity-operator
FROM golang:1.22 as builder
USER root

RUN dnf -y install libgcrypt-devel && dnf clean all && \
    dnf -y clean all && rm -rf /var/cache/dnf
    
WORKDIR /go/src/github.com/openshift/file-integrity-operator

ENV GOFLAGS="-mod=vendor"

COPY . .

RUN make build

# Step two: containerize file-integrity-operator and AIDE together
FROM registry.fedoraproject.org/fedora-minimal:37
RUN microdnf -y install aide-0.16
RUN microdnf -y install aide golang && microdnf clean all

ENV OPERATOR=/usr/local/bin/file-integrity-operator \
    USER_UID=1001 \
    USER_NAME=file-integrity-operator

# install operator binary
COPY --from=builder /go/src/github.com/openshift/file-integrity-operator/build/bin/manager ${OPERATOR}
COPY build/bin /usr/local/bin
COPY --from=builder /go/src/github.com/openshift/file-integrity-operator/build/guard/libaide_md5_guard.so /opt/libaide_md5_guard.so

RUN  /usr/local/bin/user_setup

ENTRYPOINT ["/usr/local/bin/entrypoint"]

USER ${USER_UID}
