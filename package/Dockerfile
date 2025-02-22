FROM alpine:3.18 as base
RUN apk add -U ca-certificates tar zstd tzdata
COPY build/out/data.tar.zst /
RUN mkdir -p /image/etc/ssl/certs /image/run /image/var/run /image/tmp /image/lib/modules /image/lib/firmware && \
    tar -xa -C /image -f /data.tar.zst && \
    cp /etc/ssl/certs/ca-certificates.crt /image/etc/ssl/certs/ca-certificates.crt

FROM scratch as collect
ARG DRONE_TAG="dev"
COPY --from=base /image /
COPY --from=base /usr/share/zoneinfo /usr/share/zoneinfo
RUN mkdir -p /etc && \
    echo 'hosts: files dns' > /etc/nsswitch.conf && \
    echo "PRETTY_NAME=\"K3s ${DRONE_TAG}\"" > /etc/os-release && \
    chmod 1777 /tmp

FROM scratch
COPY --from=collect / /
VOLUME /var/lib/kubelet
VOLUME /var/lib/rancher/k3s
VOLUME /var/lib/cni
VOLUME /var/log
ENV PATH="$PATH:/bin/aux"
ENV CRI_CONFIG_FILE="/var/lib/rancher/k3s/agent/etc/crictl.yaml"
ENTRYPOINT ["/bin/k3s"]
CMD ["agent"]
