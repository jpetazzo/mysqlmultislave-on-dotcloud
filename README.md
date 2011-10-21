# Deploy multiple slaves to your MySQL database on dotCloud

This is an experimental recipe to have more than 1 slave on your MySQL
database on dotCloud.

It works like this:

1. Deploy one MySQL master/slave service (as usual).
2. Add a bunch of separate MySQL standalone (non scaled) services.
3. Reconfigure those separate services as slaves of the first one.
4. Manually tweak your code to connect either to the master/slave
   service (for write ops) or to one of the standalone slaves
   (for read-only things).

This is alpha, and can probably break in many ways. If the main
master/slave service does a failover, you *might* have to manually
resync the slaves. The procedure is not complex, but it can take
some time if you have a big database.

## How to use it

Add the content of the provided dotcloud.yml file to your dotcloud.yml
build file. You don't have to include the ``dbwrite`` section if you
want to use an existing database. You can add more ``dbread`` services
if you need to.

Add the ``dbmanager`` directory to your code.

If you changed the name of the ``dbwrite`` service, or if you added more
slaves, you need to edit ``dbmanager/startsync`` to set ``MASTER`` and
``SLAVES`` environment variables.

Push.

Exec ``dotcloud run myapp.dbmanager ./startsync``.

## Todo

- guess dbwrite and dbread services by parsing ``environment.json``
- fix ``startsync`` so it waits for all servers to be ready
- run ``startsync`` automatically
- write a crude HTTP service to expose replication status
- add a long running process to check replication status, and
  optionally force a full resync 
- add a mysql proxy service to automate query routing
