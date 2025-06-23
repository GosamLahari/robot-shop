#
# Build Stage
# Using an OpenJDK 11 image as the base for the build stage.
# This avoids issues with apt-get installing Java and its certificates
# directly on a generic Debian image, as Maven requires Java.
# We choose a "slim" version to keep the build context lighter.
FROM openjdk:11-jdk-slim-buster AS build

# Update package lists and install Maven.
# Since Java is already in the base image, this should be much smoother.
RUN apt-get update && apt-get -y install maven \
    # Clean up apt caches to reduce the image size.
    && rm -rf /var/lib/apt/lists/*

# Set the working directory inside the container for the application source.
WORKDIR /opt/shipping

# Copy the Maven project file (pom.xml) to download dependencies.
COPY pom.xml /opt/shipping/
# Resolve all Maven dependencies. This step fetches all required libraries.
RUN mvn dependency:resolve
# Copy the source code into the build directory.
COPY src /opt/shipping/src/
# Package the application into a JAR file.
RUN mvn package

#
# Run Stage
# Using OpenJDK 8 for the final runtime image, as specified by your original Dockerfile.
# This stage only copies the built artifact from the 'build' stage, keeping it minimal.
FROM openjdk:8-jdk

# Expose the port that the application will listen on.
EXPOSE 8080

# Set the working directory for the running application.
WORKDIR /opt/shipping

# Define environment variables for service endpoints.
# These will be used by your Java application to connect to other services.
ENV CART_ENDPOINT=cart:8080
ENV DB_HOST=mysql

# Copy the built JAR file from the 'build' stage into the final image.
# '--from=build' specifies to copy from the stage named 'build'.
# The first path is the source path in the 'build' stage, the second is the destination in this stage.
COPY --from=build /opt/shipping/target/shipping-1.0.jar shipping.jar

# Define the command to run when the container starts.
# -Xmn and -Xmx are Java heap size settings for performance tuning.
CMD [ "java", "-Xmn256m", "-Xmx768m", "-jar", "shipping.jar" ]
