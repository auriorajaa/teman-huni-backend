FROM docker.io/python:3.12.2-slim-bookworm as python

# Stage 1: Build our dependency wheels
FROM python AS python-build-stage
ARG BUILD_ENVIRONMENT=requirements

RUN apt-get update && apt-get install --no-install-recommends -y \
  build-essential \
  libpq-dev

COPY ${BUILD_ENVIRONMENT}.txt ./

RUN pip wheel --wheel-dir /usr/src/app/wheels \
  -r ${BUILD_ENVIRONMENT}.txt

# Stage 2: Python run stage
FROM python AS python-run-stage
ARG BUILD_ENVIRONMENT=requirements
ARG APP_HOME=/app

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
ENV BUILD_ENV ${BUILD_ENVIRONMENT}

WORKDIR ${APP_HOME}

RUN apt-get update && apt-get install --no-install-recommends -y \
  sudo \
  git \
  bash-completion \
  nano \
  ssh \
  dos2unix

RUN addgroup --system django && \
  adduser --system --ingroup django django && \
  mkdir -p ${APP_HOME}/staticfiles && \
  chown django:django ${APP_HOME}/staticfiles && \
  chmod 775 ${APP_HOME}/staticfiles

RUN apt-get update && apt-get install --no-install-recommends -y \
  libpq-dev \
  gettext \
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=False \
  && rm -rf /var/lib/apt/lists/*

COPY --from=python-build-stage /usr/src/app/wheels /wheels/
RUN pip install --no-cache-dir --no-index --find-links=/wheels/ /wheels/* \
  && rm -rf /wheels/

# Copy entrypoint first and verify it exists
COPY ./docker/local/django/entrypoint.sh /entrypoint
RUN dos2unix /entrypoint && \
    chmod +x /entrypoint && \
    chown django:django /entrypoint && \
    ls -la /entrypoint

# Copy start script and verify it exists
COPY ./docker/local/django/start.sh /start
RUN dos2unix /start && \
    chmod +x /start && \
    chown django:django /start && \
    ls -la /start

# Copy application code
COPY --chown=django:django . ${APP_HOME}
RUN chown django:django ${APP_HOME}

USER django

# Use shell form for entrypoint to ensure proper execution
ENTRYPOINT ["/bin/bash", "/entrypoint"]