#
# Copy of official Java11 OpenFaaS Template
# (https://github.com/openfaas/templates/blob/master/template/java11/Dockerfile)
#
FROM openjdk:11-jdk-slim as builder

ENV GRADLE_VER=7.1.1
RUN apt-get update -qqy \
  && apt-get install -qqy \
   --no-install-recommends \
   curl \
   ca-certificates \
   unzip

RUN mkdir -p /opt/ && cd /opt/ \
    && echo "Downloading gradle.." \
    && curl -sSfL "https://services.gradle.org/distributions/gradle-${GRADLE_VER}-bin.zip" -o gradle-$GRADLE_VER-bin.zip \
    && unzip gradle-$GRADLE_VER-bin.zip -d /opt/ \
    && rm gradle-$GRADLE_VER-bin.zip

# Export some environment variables
ENV GRADLE_HOME=/opt/gradle-$GRADLE_VER/
ENV PATH=$PATH:$GRADLE_HOME/bin

RUN mkdir -p /home/app/libs

ENV GRADLE_OPTS="-Dorg.gradle.daemon=false"
WORKDIR /home/app

COPY . /home/app/

RUN gradle build
RUN find .

FROM openfaas/of-watchdog:0.7.6 as watchdog

FROM openjdk:11-jre-slim as ship
RUN apt-get update -qqy \
  && apt-get install -qqy \
   --no-install-recommends \
   unzip
RUN addgroup --system app \
    && adduser --system --ingroup app app

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

WORKDIR /home/app
COPY --from=builder /home/app/build/libs/demo-0.0.1-SNAPSHOT.jar ./demo-0.0.1-SNAPSHOT.jar
user app
#RUN unzip ./function-1.0.zip
RUN find .

WORKDIR /home/app/

ENV upstream_url="http://127.0.0.1:8082"
ENV mode="http"
ENV CLASSPATH="/home/app/demo-0.0.1-SNAPSHOT.jar:/home/app/function-1.0/lib/*"

#ENV fprocess="java -XX:+UseContainerSupport com.openfaas.entrypoint.App"
ENV fprocess="java -jar /home/app/demo-0.0.1-SNAPSHOT.jar"
EXPOSE 8080

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]