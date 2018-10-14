# Development Starter

Contains following services in form of `docker-compose` configurations:

- [MailCatcher](https://mailcatcher.me/)
- PostgreSQL v10.1 + [pgAdmin](https://www.pgadmin.org/)
- MongoDB v3.6
- Redis v4 + [Redis Commander](http://joeferner.github.io/redis-commander/)
- [Thumbor](http://thumbor.org/) v6.4.2

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
