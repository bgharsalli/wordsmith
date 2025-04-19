# Build stage
FROM maven:3-amazoncorretto-20 AS build
WORKDIR /usr/local/app
COPY pom.xml .
RUN mvn verify -DskipTests --fail-never
COPY src ./src
RUN mvn verify

# Run stage
FROM amazoncorretto:20
WORKDIR /usr/local/app
COPY --from=build /usr/local/app/target .
ENTRYPOINT ["java", "-Xmx8m", "-Xms8m", "-jar", "/usr/local/app/words.jar"]
EXPOSE 8080
