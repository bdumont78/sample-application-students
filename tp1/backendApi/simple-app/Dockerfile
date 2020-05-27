FROM openjdk:11
WORKDIR /usr/src/
COPY Main.java .
RUN javac Main.java

FROM openjdk:11
COPY --from=0 /usr/src/Main.class .
CMD java Main