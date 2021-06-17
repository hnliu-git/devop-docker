FROM openjdk:8

WORKDIR /app

RUN apt-get update && apt-get install git-core --yes

RUN cd /app && git clone https://github.com/docker-hy/spring-example-project.git

RUN cd /app/spring-example-project && ./mvnw package

EXPOSE 8080

ENTRYPOINT ["java", "-jar"]

CMD ["/app/spring-example-project/target/docker-example-1.1.3.jar"]
