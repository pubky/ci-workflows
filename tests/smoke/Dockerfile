FROM alpine:3.22

ARG TARGETARCH
RUN echo "smoke ${TARGETARCH}" > /smoke

CMD ["cat", "/smoke"]
