FROM maven:3.8.6-amazoncorretto-17 as compiler

# Build app
ENV MYAPP_HOME /opt/myapp
ENV DATABASE_HOST app-database

WORKDIR $MYAPP_HOME
COPY ./app/pom.xml .
COPY ./app/src ./src
RUN mvn package -DskipTests

# New step
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=compiler $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar