##### Elixir & Phoenix

This Dockerfile builds on the official [`elixir`](https://hub.docker.com/_/elixir/) image, adding [`Phoenix`](http://www.phoenixframework.org/) and [`nodejs`](https://nodejs.org/en/).

This image can be used either as a CLI entrypoint to the elixir ecosystem or to build and run elixir or phoenix applications.

The following aliases might be useful:

```SH
alias elixir='
  docker run \
    -it \
    --rm \
    -v "$PWD":"$PWD" \
    -w "$PWD" \
    blankenshipz/elixir-phoenix \
    elixir'

alias iex='
  docker run \
    -it \
    --rm \
    -v "$PWD":"$PWD" \
    -w "$PWD" \
    blankenshipz/elixir-phoenix'

alias mix='
  docker run \
    -it \
    --rm \
    -v "$PWD":"$PWD" \
    -w "$PWD" \
    blankenshipz/elixir-phoenix \
    mix'
```

### Using this image for development

Here's an example Dockerfile and `docker-compose.yml` that will work with the phoenix `hello_phoenix` [demo application](http://www.phoenixframework.org/docs/up-and-running).

```SH
# Dockerfile

FROM blankenshipz/elixir-phoenix

RUN \
  apt-get update && \
  apt-get install -y postgresql-client && \
  apt-get clean && \
  cd /var/lib/apt/lists && rm -fr *Release* *Sources* *Packages* && \
  truncate -s 0 /var/log/*log

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

COPY mix.exs /usr/src/app/
COPY mix.lock /usr/src/app/
COPY package.json /usr/src/app/

RUN \
  mix local.hex --force && \
  mix deps.get && \
  npm install --quiet

COPY . /usr/src/app/

CMD ["mix", "phoenix.server"]
```

And the `docker-compose.yml`: 

```SH
# docker-compose.yml
version: '2'

services:
  web: &default
    build: .
    command: "mix phoenix.server"
    ports:
      - "4000:4000"
    links:
      - postgres
    volumes:
      - .:/usr/src/app

  postgres:
    image: postgres
    ports:
      - "5432:5432"
```
