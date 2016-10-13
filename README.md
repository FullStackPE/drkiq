Dockerizing a Ruby on Rails Application
=======================================

Ref: [Dockerizing a Ruby on Rails Application](https://semaphoreci.com/community/tutorials/dockerizing-a-ruby-on-rails-application)

---------------------------------------------------------------

# Brief version

- Install `docker`
- Install `docker-compose`
- Create `postgres` and `redis` volumes
- Create and migrate db using `rake` commands with `docker-compose`
- Run `docker-compose up`

# Linux setup

## Ubuntu
Ref: [https://docs.docker.com/engine/installation/linux/ubuntulinux/](https://docs.docker.com/engine/installation/linux/ubuntulinux/)

- Clone the project:

        git clone https://github.com/FullStackPE/drkiq.git
        cd drkiq

- Check your Ubuntu version. It has to be 64-bit. Additionally, your kernel must be 3.10 at minimum:

        uname -r

- Ensure that CA certificates are installed, and add the new GPG key

        sudo apt-get install apt-transport-https ca-certificates
        sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

- Add the following line to `/etc/apt/sources.list.d/docker.list`. The last part of the command depends on your Ubuntu version:

        deb https://apt.dockerproject.org/repo ubuntu-trusty main

- Update and verify that APT is pulling from the right repository:

        sudo apt-get update
        apt-cache policy docker-engine

- Install Docker Compose ([reference 1](https://docs.docker.com/compose/install), [reference 2](https://github.com/docker/compose/releases)):

        sudo -i
        curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
        chmod +x /usr/local/bin/docker-compose
        sudo apt-get update
        sudo apt-get install docker-engine

- Try to start docker. It's supposed to fail (almost always), so don't panic:

        service docker start

  If the last command failed, use `sudo` in front of it ([reference](https://github.com/docker/compose/issues/1214)):

      sudo service docker start

- Create `postgres` and `redis` volumes:

        sudo docker volume create --name drkiq-postgres
        sudo docker volume create --name drkiq-redis

- Let's try to run everything. Spoiler: it will fail!

        sudo docker-compose up

  Since the last command probably failed, please check the "Errors" section below. Once you solved the problem, run `sudo docker-compose up` again.

- You should now notice a few errors related to the db. That's because we still didn't create it. So let's stop everything (`CTRL C`) and set up the db:

        sudo docker-compose run --user "$(id -u):$(id -g)" drkiq rake db:reset
        sudo docker-compose run --user "$(id -u):$(id -g)" drkiq rake db:migrate

- In your browser, you should find everything at `http://localhost:8000`.

### Errors

- You might get some error related to open ports:

  > ERROR: for postgres  Cannot start service postgres: driver failed programming external connectivity on endpoint drkiq_postgres_1 (39c02de08efcf3c9b4670ba0158f3f5bbc81324fb969d2043e77a2cf7ba90b6e): Error starting userland proxy: listen tcp 0.0.0.0:5432: bind: address already in use
  > ERROR: Encountered errors while bringing up the project.

  Find out which process is using that door:

      sudo lsof -i :5432

  So we'll find out that `postgres` is using that port. Just shut down `postgres`:

      sudo /etc/init.d/postgresql stop

  Same for the other errors. For example, with `redis`:

      sudo lsof -i :6379
      # Guess what: it's redis-server!
      sudo /etc/init.d/redis-server stop

  Then we can give again:

      sudo docker-compose up

# Mac OS X setup

Donwload and install the stable version of docker for mac as described here:

https://docs.docker.com/docker-for-mac

- Clone the project:

    `git clone https://github.com/FullStackPE/drkiq.git`

    `cd drkiq`

- Start the containers

    `docker-compose up`

- You may need to create and migrate the db

  `docker-compose run drkiq rake db:reset`

  `docker-compose run drkiq rake db:migrate`

## Errors

To solve problems related to stopping local services that bind to the same ports used by Postgres and Redis (that run inside the container).

```
ERROR: for postgres  Cannot start service postgres: driver failed programming external connectivity on endpoint drkiq_postgres_1 (cb77f86584efd7f70c799b8f835fd9eb2ef257a7773d1bdd746375ad6a7e4798): Error starting userland proxy: Bind for 0.0.0.0:5432: unexpected error Uwt.Uwt_error(Uwt.EADDRINUSE, "listen", "")
ERROR: Encountered errors while bringing up the project.
```

Identify the service who bind to `5432` port

`lsof -i :5432`

```
COMMAND  PID USER  FD   TYPE DEVICE SIZE/OFF NODE NAME
postgres 99 johnny 5u   IPv6 0x29      0t0   TCP  localhost:postgresql (LISTEN)
```

`brew services stop postgres`

The same applies to Redis service
