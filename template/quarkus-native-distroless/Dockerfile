# Get a build environment
FROM softinstigate/graalvm-maven:latest as build-aot
# Set working dir
RUN mkdir /app && mkdir -p /tmp/tmpapp
WORKDIR /app

COPY settings.xml /root/.m2/settings.xml

# Cache maven dependency
ADD function/pom.xml /tmp/tmpapp/pom.xml
RUN mvn -f /tmp/tmpapp/pom.xml dependency:resolve

# build the function
ADD function /app/function
RUN mvn -f /app/function/pom.xml package -Pnative

# use the openfaas image to get the watchdog
FROM openfaas/of-watchdog:0.8.2 as watchdog

# Create new image
FROM quay.io/quarkus/quarkus-micro-image:1.0

COPY --from=build-aot /app/function/target/classes/*.properties /config/
COPY --from=build-aot /app/function/target/*-runner /function
COPY --from=watchdog /fwatchdog /fwatchdog

EXPOSE 8080
USER 1001

ENV cgi_headers="true"
ENV fprocess="/function"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:8000"

ENV exec_timeout="20s"
ENV write_timeout="25s"
ENV read_timeout="25s"

HEALTHCHECK --interval=2s CMD ["/fwatchdog", "healthcheck"]

CMD ["/fwatchdog"]
