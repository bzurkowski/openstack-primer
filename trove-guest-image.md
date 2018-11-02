# Trove guest image setup

## Building image

Add non-root user:

```bash
$ adduser stack
$ usermod -aG sudo stack
```

Switch to new user account:

```bash
$ su - stack
```

Install Disk Image Builder following the [instructions](disk-image-builder.md).

Activate virtualenv:

```bash
$ . ~/.venvs/dib/bin/activate
```

Clone Trove repository:

```bash
$ https://github.com/openstack/trove.git
```

Run kick-start command:

```bash
$ ./trove/integration/scripts/trovestack build-image mysql
```

Guest image will be built into `/home/stack/images` directory.


## Loading image into Trove

Upload image to Glance:

```bash
$ openstack image create ...
```

Setup datastore in Trove:

```bash
DATASTORE_TYPE=mysql
VERSION=5.7
IMAGEID=c84cfa29-03eb-4704-b31a-a3ce3202c9fb
PACKAGES=mysql-server-5.7
PATH_TROVE=<Path to cloned Trove repository>
```

```bash
trove-manage datastore_update "$DATASTORE_TYPE" ""
trove-manage datastore_version_update "$DATASTORE_TYPE" "$VERSION" "$DATASTORE_TYPE" $IMAGEID "$PACKAGES" 1
trove-manage datastore_update "$DATASTORE_TYPE" "$VERSION"
trove-manage db_load_datastore_config_parameters "$DATASTORE_TYPE" "$VERSION" "$PATH_TROVE"/trove/templates/$DATASTORE_TYPE/validation-rules.json
````
