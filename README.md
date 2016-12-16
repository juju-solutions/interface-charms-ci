# Overview

This interface layer handles the communication between the Cloud Weather
Report CI system and a client (e.g. Review Queue).


# Usage

## Provides

The [cwr][] charm is an example provider of this interface. This charm signals
to all its related clients when it becomes ready.

[cwr]: https://github.com/juju-solutions/layer-cwr

The interface layer sets the following state as soon as a client is connected:

  * `{relation_name}.joined`: A client has been related to CWR.

CWR can signal information about its availability with the following methods:

  * `set_ready()`: CWR has a port and controller data and is ready.

  * `clear_ready()`: CWR is not ready.

An example of a charm providing this interface would be:

```python
@when('cwr.started', 'client.joined')
def client_present(client):
    if port and controllers:
        client.set_controllers(controllers)
        client.set_port(port)
        client.set_rest_prefix('/ci')
        client.set_ready()
    else:
        client.clear_ready()

@when('client.joined')
@when_not('cwr.started')
def client_should_stop(client):
    client.clear_ready()
```

## Requires

This is the side of the interface that a client charm will use to be informed
of the availability of CWR. The [review-queue][] is an example charm that
requires this interface.

[review-queue]: https://github.com/juju-solutions/review-queue-charm

The interface layer sets the following states for the client to react to:

  * `{relation_name}.joined`: The client has been related to CWR and is waiting
  for CWR to become ready.

  * `{relation_name}.ready`: CWR is ready.

  * `{relation_name}.store.ready`: CWR is ready and has been provided a token
  that allows it to release charms to the store.

The client can retrieve information from CWR when ready:

  * `get_rest_url`: Returns `http://ip:port/path` REST URL for CWR.

  *  `[controllers|port|rest_prefix|store_token]()`: Returns remote data.

An example of a charm using this interface would be:

```python
@when('cwr.store.ready')
@when_not('revq.started')
def start(cwr):
    rest = cwr.get_rest_url()
    controllers = cwr.controllers()
    token = cwr.store_token()
    start_revq(rest, controllers, token)
    set_state('revq.started')

@when('revq.started')
@when_not('cwr.ready')
def stop():
    stop_revq()
    remove_state('revq.started')
```


# Resources

- [Juju mailing list](https://lists.ubuntu.com/mailman/listinfo/juju)
- [Juju community](https://jujucharms.com/community)
