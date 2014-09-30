nginx-proxy sets up a container running nginx and [docker-gen][1].  docker-gen generate reverse proxy configs for nginx and reloads nginx when containers they are started and stopped.

See [Automated Nginx Reverse Proxy for Docker][2] for why you might want to use this.

### Usage

To build it:

    $ docker build -t camomile/nginx-proxy .

To run it:

    $ docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock camomile/nginx-proxy

Then start any container you want proxied with env vars `VIRTUAL_HOST=subdomain.youdomain.com` and `VIRTUAL_PATH=location`

    $ docker run -e VIRTUAL_HOST=foo.bar.com -e VIRTUAL_PATH=baz ...

Provided your DNS is setup to forward `foo.bar.com` to the a host running nginx-proxy, a request to `foo.bar.com/baz` will be routed to the container with the matching `VIRTUAL_HOST` and `VIRTUAL_PATH` env vars set.

### Multiple Ports

If your container exposes multiple ports, nginx-proxy will default to the service running on port 80.  If you need to specify a different port, you can set a `VIRTUAL_PORT` env var to select a different one.  If your container only exposes one port and it has a `VIRTUAL_HOST` env var set, that port will be selected.

  [1]: https://github.com/jwilder/docker-gen
  [2]: http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/

### Multiple Hosts

If you need to support multipe virtual hosts for a container, you can separate each entry with commas.  For example, `foo.bar.com,baz.bar.com,bar.com` and each host will be setup the same.


### Camomile setup

#### nginx proxy

    $ docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock camomile/nginx-proxy

#### Camomile REST API (at camomile.fr/api)

    $ docker run -d \
      -v /path/to/the/database:/data/db \                   # Database is stored locally in /path/to/the/database 
      --name mongodb dockerfile/mongodb
                    
    $ docker run -d \
      -e VIRTUAL_HOST=camomile.fr -e VIRTUAL_PATH=api \     # nginx-proxy redirects from camomile.fr/api
      -e ROOT_PASSWORD=R00t.p455w0rd                        # API root password is R00t.p455w0rd 
      -v /path/to/media:/media \                            # Media files are stored locally in /path/to/media
      --link mongodb:mongodb \                              # API is linked to this MongoDB instance
      --name camomile_api camomile/api                 

#### Camomile Touch Web App (at camomile.fr/touch)

    $ docker run -d \
      -e VIRTUAL_HOST=camomile.fr -e VIRTUAL_PATH=touch \   # nginx-proxy redirects from camomile.fr/touch
      -e CAMOMILE_API=camomile.fr/api \                     # Web app speaks with API at camomile.fr/api 
      --name camomile_touch camomile/touch

#### Camomile Admin Web App (at camomile.fr/admin)

    $ docker run -d \
      -e VIRTUAL_HOST=camomile.fr -e VIRTUAL_PATH=admin \   # nginx-proxy redirects from camomile.fr/admin
      -e CAMOMILE_API=camomile.fr/api \                     # Wep app speaks with API at camomile.fr/api
      --name camomile_admin camomile/admin
