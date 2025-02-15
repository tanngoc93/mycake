ARG RAILS_MASTER_KEY

FROM ruby:2.7.5

ENV PORT=3000 \
    RAILS_ENV=development \
    APP_DIR=/home/app/html \
    BUNDLE_PATH=/bundle/vendor \
    RAILS_MASTER_KEY=$RAILS_MASTER_KEY

RUN apt-get update -qq \
  && apt-get install -yq --no-install-recommends \
    build-essential \
    gnupg2 \
    curl \
    less \
    git \
    graphviz \
  && apt-get clean \
  && rm -rf /var/cache/apt/archives/* \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log

RUN mkdir -p $APP_DIR $APP_DIR/log $APP_DIR/tmp/pids

RUN apt-get update -qq && apt-get install -y postgresql-client
RUN apt-get install -y shared-mime-info

COPY Gemfile      $APP_DIR/Gemfile
COPY Gemfile.lock $APP_DIR/Gemfile.lock

# Application dependencies
# External Aptfile for that
COPY ./.dockerdev/Aptfile /tmp/Aptfile
RUN apt-get update -qq && apt-get -yq dist-upgrade && \
    apt-get install -yq --no-install-recommends \
    $(grep -Ev '^\s*#' /tmp/Aptfile | xargs) && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    truncate -s 0 /var/log/*log

# Configure bundler
ENV LANG=C.UTF-8 \
  BUNDLE_JOBS=4 \
  BUNDLE_RETRY=3

# Store Bundler settings in the project's root
ENV BUNDLE_APP_CONFIG=.bundle

# Upgrade RubyGems and install latest Bundler
RUN gem update --system && \
    gem install bundler

WORKDIR $APP_DIR
COPY .  $APP_DIR/

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

#
EXPOSE $PORT

# Configure the main process to run when running the image
CMD ["rails", "server", "-b", "0.0.0.0"]
