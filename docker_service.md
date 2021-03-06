This document states about the necessary elements that need to be considered when we develop a new docker/micro service.

# Dockerfile
  1. Mention specific version of base image. Avoid using latest images.
  2. Use one RUN to prepare, configure, make, install and cleanup. If the lines get too long, put the steps in separate scripts and call them.
  3. Install specific version of open source libraries and dependent libraries. Helps to check OSS clearance of specific versions.
  4. Use multi-staging to create smallest possible images. Several steps can help to acheive this-
    a. Choose the base image for final stage wisely.
    b. Try your best to use less RUN commands. Scripts can be used to reduce number of RUN commands.
    c. Clean up the installation of libraries that are automatically installed, when you install specific libraries. There are different ways based on platform. 
        We must make sure that dependent libraries, cache and libraries installed with "update" are also cleaned-up during clean-up of libraries.
        Please refer to below links for reference -
          alpine: virtual packaging approach can be used to cleanup all resources
            https://stackoverflow.com/questions/46221063/what-is-build-deps-for-apk-add-virtual-command
        debian: Use apt-mark showauto`, purge and perform clean-up of apt
            https://www.dajobe.org/blog/2015/04/18/making-debian-docker-images-smaller/ 
  5. Change USER to non-root user(with a specific ID) after performing necessary steps. Services with non-root user access has less security attack surface.
  6. if a service require access to HOST system, required access level can be provided to docker user (Point 5) using init container.
  7. Provide only necessary access permissions to files in docker image.
  8. avoid using environment variables. 
  9. Do not mention passwords or critical  information in Dockerfile
  10. Create a WORKDIR, to refer to required executables during docker run
  11. EXPOSE container port

# Health Check
  Develop health check for service based on the nature of service. Build API's as required to provide health status of service.
  An explicit command or script shall be chosen to check the health status of service.
  
  This enables us to use it in docker-compose environment & Kubernetes (Liveness, Readiness & Startup) to stream line start-up sequence, monitoring health of service and scalability aspects.
  For eg: GRPC supports grpc_health_probe for health check of a grpc server.
          Postgres provides pg_isready to check the availability of database [HEALTHCHECK can be seen as a part of Dockerfile itself]

# Fault tolerance
  When a service is dependent on other services, make sure that there is a retry mechanism in place based on the nature of dependency to send requests to dependent service.
  we must make sure that there is no cascading effect of the retry mechanism on the overall application or dependent services.
  
  This helps to handle scenario's, when dependent service is started with a delay or is not available for sometime.
  
# Crash Recovery
  When a service is crashed due to programmatic errors or recource limitations, make sure that those services are started successfully.
  At times, if clean-up has not happened competely, service will not start correctly. Make sure that you have approaches in place to avoid this.

  eg: "port already in use" kind of issues.

# Graceful shutdown
  Make sure that your service is shutdown gracefully when User performs "docker stop" or "docker-compose stop". 
  By default "SIGTERM" is invoked on container, if no explicit signal is mentioned.

  We can explicitly mention "STOP SIGNAL" in Dockerfile or "docker run" to generate a specific signal when a service is to be stopped.
  So, Application can either catch "SIGTERM" or a specific signal to terminate required services and perform clean-up on "SIG-TERM".
  
  For eg: Postgres takes "SIGINT" as STOP SIGNAL.
  
  This helps to proper clean-up of resources and also quick shutdown of the service. Otherwise, docker daemon waits for 10sec(by default) to stop the container.
  Implicitly it may help for faster start-up of the service as well.
  
# Logging
  Choose to have a logging approach for your service, in case if there are no central logging service available.
  Make sure that different log levels are used. This enables us to enable logging for specific set of logs in DEV & PROD environments.
  
# Reduce build time of images and OSS upgrades across images
  Build small images with the required libraries and use them as base images to reduce build time. For eg: gRPC, Protobuf, numpy, pandas, grpcio.
  Since the versions of libraries doesn't change everyday, this helps to reduce the build time of the image.

# Versioning of API
  Version your API's, so that any modifications can be easily handled and usable for clients without downtime.
  
# Versioning of Database storage
  Use versioning for your database schema for seemless modification of schema when required.
  
# Disaster Recovery
