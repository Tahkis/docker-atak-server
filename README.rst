===================
Run TAK Java server
===================

tldr::

    cp takserver.env.example takserver.env
    # edit the env
    # export the variables gomplate uses (see the .tpl files)
    cat docker-compose.yml.tpl | gomplate >docker-compose.yml
    cat traefik.toml.tpl | gomplate >traefik.toml
    docker-compose -p tak up -d

or use docker-compose.local.yml without gomplate for local dev (rebuilding containers)::

    docker build --progress=plain --target production -t takserver:certsapi-latest -t pvarkiprojekti/takserver:certsapi-latest -f python-takcertsapi/Dockerfile ./
    docker build --progress=plain -t takserver:latest -t takserver:4.7-RELEASE-32 -t pvarkiprojekti/takserver:4.7-RELEASE-32 .
    cp takserver.env.example takserver.env
    # edit the env
    docker-compose -f docker-compose.local.yml -p tak up

Note, for things that live in the volumes (like TAK certs) you must nuke the volumes to see changes::

    docker-compose -f docker-compose.local.yml -p tak down -v ; docker-compose -f docker-compose.local.yml -p tak rm -vf



Creating client packages locally
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Using the REST API is probably nicer though

Create client package::

    docker-compose -p tak exec takserver /bin/bash -c 'CLIENT_CERT_NAME=replaceme /opt/scripts/make_client_zip.sh'

Then get /opt/tak/certs/files/clientpkgs/replaceme.zip out of the container::

    docker-compose -p tak exec takserver /bin/bash -c 'base64 /opt/tak/certs/files/clientpkgs/replaceme.zip' | base64 -id >replaceme.zip

This approach also works for recovering the admin cert (/opt/tak/certs/files/admin.p12 unless you changed the ADMIN_CERT_NAME ENV)


Creating new admin users
^^^^^^^^^^^^^^^^^^^^^^^^

Create the user on the takserver container::

    docker-compose -p tak exec takserver /bin/bash -c 'cd /opt/tak/data/certs/ && CAPASS=$CA_PASS PASS=replaceme_user_cert_pass ./makeCert.sh client replaceme_username && ADMIN_CERT_NAME=replaceme_username /opt/scripts/enable_admin.sh'

See above about the hard way of getting the cert file, or use the REST API.


Gradle builds
^^^^^^^^^^^^^

Build the distribution::

    mkdir outputs
    docker build --progress=plain -f Dockerfile_build --target files -t atakbuild:files  .
    docker run --rm -it -v `pwd`/outputs:/output atakbuild:files

Now you have the build artefacts in outputs -directory.
