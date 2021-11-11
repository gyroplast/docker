FROM alpine:latest as build

RUN \
    apk update &&\
    apk add --no-cache gcc musl-dev openssl-dev libcap-dev zlib-dev zlib-static make binutils cvs

RUN mkdir -p /build && cd /build
WORKDIR /build

RUN cvs -d :pserver:cvs@cvs.fefe.de:/cvs co libowfat
RUN cvs -d :pserver:cvs@cvs.fefe.de:/cvs co gatling

RUN \
    cd /build/libowfat &&\
    make DIET= NATIVE="-march=native -mtune=native"
    
RUN \
    cd /build/gatling &&\
    sed -i 's,^\s*#define SUPPORT_\(SMB\|FTP\|CGI\),// \0,' gatling_features.h &&\
    make DIET= PIE=1 install


FROM alpine:latest

LABEL org.opencontainers.image.authors="kontakt@veloxis.de"

COPY --from=build /opt/diet/bin/gatling /bin/gatling

RUN \
    addgroup -g 1000 httpd &&\
    adduser -h /srv/www -s /sbin/nologin -D -G httpd -u 1000 httpd

EXPOSE 8000
VOLUME /srv/www

ENTRYPOINT ["gatling", "-c", "/srv/www", "-u", "1000", "-p", "8000"]
CMD ["-V", "-D", "-n", "-L"]
