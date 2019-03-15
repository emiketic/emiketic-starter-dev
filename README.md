# Starter Development

![](https://img.shields.io/david/emiketic/emiketic-starter-dev.svg?style=for-the-badge)

A set of common development tooling.

Contains following services in form of `docker-compose` configurations:

- PostgreSQL v10.1 + [pgAdmin](https://www.pgadmin.org/)
- MongoDB v4.0
- Redis v5.0 + [Redis Commander](http://joeferner.github.io/redis-commander/)
- ElasticSearch v6.6
- [Minio](https://www.minio.io/)
- [Thumbor](http://thumbor.org/) v6.4.2
- [MailCatcher](https://mailcatcher.me/)
- [Notification Catcher](https://github.com/notifme/catcher)

Contains also a `Vagrant` configuration for ready-to-use development VM for Node.js, Meteor and Ruby/Rails.

## Requirements

- [Docker v17.\*](https://docs.docker.com/engine/installation/linux/ubuntu/#install-docker)
- [Docker Compose v1.\*](https://docs.docker.com/compose/install/#alternative-install-options)
- [Vagrant v1.\*](https://www.vagrantup.com/)

## Usage

### Services

Services can be run collectively or individually:

```sh
# start all services
docker-compose up

# start a particular service
cd ./SERVICE
docker-compose up

# stop services
# Control+C
```

### VM

```sh
# start
vagrant up

# ssh
vagrant ssh

# stop
vagrant stop
```

## [Documentation](./docs)
