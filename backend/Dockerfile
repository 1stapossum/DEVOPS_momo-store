FROM alpine:3.12.3

#RUN apk add --no-cache iputils busybox-extras curl
ARG VERSION=${VERSION}

RUN adduser -D -g '' appuser

WORKDIR /app

COPY --chown=appuser:appuser {CI_PROJECT_DIR}/backend/momo-store-${VERSION}/api-${VERSION} api

USER appuser
EXPOSE 8081
CMD ["/api"]