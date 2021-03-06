FROM ruby:2.6.5-alpine
LABEL maintainer="georg@ledermann.dev"

# Add basic packages
RUN apk add --update --no-cache --virtual build-dependencies\
      build-base \
      postgresql-dev \
      git \
      nodejs \
      yarn \
      tzdata \
      file \
      python \
      make \
      g++ \
      && npm install \
      && apk del build-dependencies

WORKDIR /app

# Install standard Node modules
COPY package.json yarn.lock /app/
RUN yarn install

# Install standard gems
COPY Gemfile* /app/
RUN bundle config --global frozen 1 && \
    bundle config --local build.sassc --disable-march-tune-native && \
    bundle install -j4 --retry 3

#### ONBUILD: Add triggers to the image, executed later while building a child image

# Install Ruby gems (for production only)
ONBUILD COPY Gemfile* /app/
ONBUILD RUN bundle install -j4 --retry 3 --without development:test && \
  # Remove unneeded gems
  bundle clean --force && \
  # Remove unneeded files from installed gems (cached *.gem, *.o, *.c)
  rm -rf /usr/local/bundle/cache/*.gem && \
  find /usr/local/bundle/gems/ -name "*.c" -delete && \
  find /usr/local/bundle/gems/ -name "*.o" -delete

# Copy the whole application folder into the image
ONBUILD COPY . /app

# Compile assets with webpacker
ONBUILD RUN RAILS_ENV=production \
            SECRET_KEY_BASE=dummy \
            bundle exec rails assets:precompile

# Remove folders not needed in resulting image
ONBUILD RUN rm -rf node_modules tmp/cache vendor/bundle test spec
