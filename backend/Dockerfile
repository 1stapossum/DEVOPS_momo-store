FROM gcr.io/distroless/base-debian10
ARG VERSION=${VERSION}
WORKDIR /
COPY momo-store-${VERSION}/api-${VERSION} /api
EXPOSE 8081
USER nonroot:nonroot
ENTRYPOINT ["/api"]




