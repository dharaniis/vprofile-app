# ── Stage 1: Build ────────────────────────────────────────────────────────────
FROM maven:3.9.9-eclipse-temurin-21-jammy AS BUILD_IMAGE

WORKDIR /app

# Copy only the dependency descriptor first so this layer is cached
# as long as pom.xml doesn't change.
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy the rest of the source and produce the WAR
COPY src ./src
RUN mvn clean package -DskipTests -B

# ── Stage 2: Runtime ──────────────────────────────────────────────────────────
FROM tomcat:10-jdk21

# Remove the default ROOT webapp
RUN rm -rf /usr/local/tomcat/webapps/*

# Drop the built WAR in as ROOT.war so Tomcat serves it at /
COPY --from=BUILD_IMAGE /app/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]
