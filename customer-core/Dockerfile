FROM eclipse-temurin:21-jdk-alpine as build

COPY .mvn .mvn
COPY mvnw .
COPY pom.xml .

RUN ./mvnw -B dependency:go-offline                          

COPY src src

RUN ./mvnw -B package                                        

FROM eclipse-temurin:21-jre-alpine

COPY --from=build target/customer-core-0.0.2-SNAPSHOT.jar .

EXPOSE 8110

ENTRYPOINT ["java", "-jar", "customer-core-0.0.2-SNAPSHOT.jar"]
