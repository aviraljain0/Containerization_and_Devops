FROM postgres:15-alpine

# Fulfilling the requirement to configure DB parameters in the Dockerfile
# These act as defaults but will be overridden securely by docker-compose
ENV POSTGRES_DB=appdb
ENV POSTGRES_USER=appuser
ENV POSTGRES_PASSWORD=apppass

# The official Postgres image drops root privileges automatically and runs as the 'postgres' user.
# Adding custom configuration files or initialization scripts can be done here.
RUN mkdir -p /custom-init

# Healthcheck configuration inside the Dockerfile (optional layer optimization)
HEALTHCHECK --interval=10s --timeout=5s --retries=5 \
  CMD pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB} || exit 1