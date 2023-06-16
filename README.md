This repository contains an example configuration for a deployment of the
[Matrix Synapse][synapse-repo] server with multiple worker processes.

I put this together with reference to the [canonical documentation for Synapse
workers][workers-docs], the [Synapse configuration guide][config-guide] and
help from the [Synapse Admins room on matrix.org][synapse-room].

Note you should not attempt to set up a Synapse server and workers without
reading the above documentation.  However if you need some additional context
or examples to understand what the documentation is telling you, the example
files here may be useful.

The intended configuration is summarised in the image below:

![Synapse workers architecture diagram](images/synapse-workers-architecture.png)

The main homeserver process is supplemented with six worker processes:

- 3 generic workers to handle a variety of incoming client requests
- 1 events persister worker to handle writing the events stream
- 1 receipts writer worker to handle writing the `receipts` stream and the
  `typing` stream
- 1 federation sender to handle outgoing traffic to other servers

The Nginx server is configured to proxy requests though to different workers or
to the main process based on pattern matching the URL path of the request.

Each worker might be listening on a port in the 8xxx range for requests
received via Nginx.  In this example, the `events_persister1` worker has a port
allocation of 8111 that is greyed out in the diagram because this worker is not
set up to handle any requests from Nginx and isn't actually listening on that
port.

Additionally, each worker might listen on another port in the 9xxx range for
"replication" requests from other workers.  The `generic_worker`s in the diagram
have these ports greyed out because they are not set up to handle any requests
from other workers.

Note these port ranges are completely arbitrary - you can choose port numbers
that work for you.  It's important to remember that the worker replication ports
(the 9xxx ones in my example) should never be exposed via Nginx and should only
accept connections from other local workers.

A worker would refer to the [`stream_writers`][stream-writers-docs]
configuration (in [`homeserver.yaml`][homeserver-yaml]) to determine which
worker is responsible for handling a particular stream.  Then it would refer to
the [`instance_map`][instance-map-docs] configuration (also in
[`homeserver.yaml`][homeserver-yaml]) to determine which port number the named
worker is listening on.  A worker knows which port to listen on based on the
individual worker config file - so the port number listed in the worker config
must match the port number in the `instance_map`.

In this repo you'll find:

- a fairly complete [homeserver.yaml](etc/matrix-synapse/homeserver.yaml)
- a set of [config files for workers](etc/matrix-synapse/workers)
- a set pf [systemd unit files](etc/systemd/system) for managing the main and
  worker processes
- an [Nginx config](etc/nginx/nginx.conf) for mapping requests to worker ports
  based on the request URL path


[synapse-repo]: https://github.com/matrix-org/synapse/
[workers-docs]: https://matrix-org.github.io/synapse/latest/workers.html
[config-guide]: https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html
[synapse-room]: https://app.element.io/#/room/#synapse:matrix.org
[stream-writers-docs]: https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html?highlight=stream_writers#stream_writers
[homeserver-yaml]: etc/matrix-synapse/homeserver.yaml
[instance-map-docs]: https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html?highlight=instance_map#instance_map
