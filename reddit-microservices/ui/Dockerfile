# FROM ruby:2.2
# RUN apt-get update -qq && apt-get install --no-install-recommends -y build-essential && rm -rf /var/lib/apt/lists/*

# ENV APP_HOME /app
# RUN mkdir "$APP_HOME"
# WORKDIR $APP_HOME

# # ADD Gemfile* $APP_HOME/
# COPY . $APP_HOME
# RUN bundle install

# ENV POST_SERVICE_HOST post
# ENV POST_SERVICE_PORT 5000
# ENV COMMENT_SERVICE_HOST comment
# ENV COMMENT_SERVICE_PORT 9292

# CMD ["puma"]

# FROM ubuntu:16.04

# RUN apt-get update \
# && apt-get install --no-install-recommends -y ruby-full ruby-dev build-essential \
# && gem install bundler --no-ri --no-rdoc \
# && rm -rf /var/lib/apt/lists/*

# ENV APP_HOME /app
# RUN mkdir "$APP_HOME"
# WORKDIR $APP_HOME

# COPY . $APP_HOME
# RUN bundle install

# ENV POST_SERVICE_HOST post
# ENV POST_SERVICE_PORT 5000
# ENV COMMENT_SERVICE_HOST comment
# ENV COMMENT_SERVICE_PORT 9292

# CMD ["puma"]

FROM alpine:latest

ENV POST_SERVICE_HOST post
ENV POST_SERVICE_PORT 5000
ENV COMMENT_SERVICE_HOST comment
ENV COMMENT_SERVICE_PORT 9292
ENV APP_HOME /app

RUN mkdir "$APP_HOME"
WORKDIR $APP_HOME
COPY . $APP_HOME

RUN apk update \
 && apk add --no-cache build-base \
                       ruby \
                       ruby-bundler \
                       ruby-dev \
                       ruby-json \
 && gem install bundler --no-ri --no-rdoc \
 && bundle install && apk del build-base ruby-dev \
 && rm -rf /usr/share/terminfo/ \
 && rm -rf /root/.bundle/cache/

CMD ["puma"]
