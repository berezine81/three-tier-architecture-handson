#
# Build
#
FROM maven:3.8.4-openjdk-17 AS build

WORKDIR /opt/shipping

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src/ /opt/shipping/src/
RUN mvn package -DskipTests


FROM openjdk:17-jdk-slim

EXPOSE 8080

WORKDIR /opt/shipping

ENV CART_ENDPOINT=cart:8080
ENV DB_HOST=mysql

COPY --from=build /opt/shipping/target/shipping-1.0.jar shipping.jar

CMD ["java", "-Xmn256m", "-Xmx768m", "-jar", "shipping.jar"]
