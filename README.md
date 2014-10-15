# ianblenke/newrelic

Run the New Relic server monitor daemon for docker and coreos server.

This is the [ianblenke/newrelic](https://github.com/uzyexe/dockerfile-newrelic) github project, which backs the [ianblenke/newrelic](https://index.docker.io/u/ianblenke/newrelic) docker hub image.

This is a _fork_ of the [uzyexe/dockerfile-newrelic](https://github.com/uzyexe/dockerfile-newrelic) github project, which backs the [uzyexe/newrelic](https://index.docker.io/u/uzyexe/newrelic) docker hub image.

## Dockerfile

[**Trusted Build**](https://index.docker.io/u/ianblenke/newrelic)

This Docker image is based on the official [debian:squeeze](https://index.docker.io/_/debian/) base image.

## Using

**Please note: Replaced by your newrelic license key is `YOUR_NEW_RELIC_LICENSE_KEY`**

### case 1: docker run


    docker run -d -e NEW_RELIC_LICENSE_KEY=YOUR_NEW_RELIC_LICENSE_KEY -h `hostname` ianblenke/newrelic

--

### case 2: Auto-Running configure for cloud-config.yml (for Disk Booting coreos)


    coreos:
      units:
        - name: docker.service
          command: start
        - name: newrelic-client.service
          command: start
          content: |
              [Unit]
              Description=newrelic-client
              
              [Service]
              Restart=always
              TimeoutStartSec=20m
              ExecStartPre=-/usr/bin/docker rm -f newrelic-client
              ExecStart=/bin/bash -c 'HOSTNAME=`/usr/bin/hostname`; docker run --name newrelic-client --rm --env="NEW_RELIC_LICENSE_KEY=YOUR_NEW_RELIC_LICENSE_KEY" -h $HOSTNAME uzyexe/newrelic'
              ExecStop=/usr/bin/docker kill newrelic-client

--

### case 3: Auto-Running configure for cloud-config.yml (for PXE Booting coreos)

**Please note: Replaced by your newrelic license key is `YOUR_NEW_RELIC_LICENSE_KEY`**


    coreos:
      units:
        - name: docker.service
          command: restart
          content: |
              [Unit]
              Description=Docker Application Container Engine
              Documentation=http://docs.docker.io
              
              [Service]
              Environment="TMPDIR=/var/tmp/"
              ExecStartPre=/bin/mount --make-rprivate /
              ExecStart=/usr/bin/docker -d -r=false -H fd://
              
              [Install]
              WantedBy=multi-user.target
        - name: newrelic-client.service
          command: start
          content: |
              [Unit]
              Description=newrelic-client
              
              [Service]
              Restart=always
              TimeoutStartSec=20m
              ExecStart=/bin/bash -c 'HOSTNAME=`/usr/bin/hostname`; docker run --name newrelic-client --rm --env="NEW_RELIC_LICENSE_KEY=YOUR_NEW_RELIC_LICENSE_KEY" -h $HOSTNAME ianblenke/newrelic'
              ExecStop=/usr/bin/docker kill newrelic-client
    
    write_files:
      - path: /tmp/coreos-setup-hostname
        content: |
            #!/bin/bash +x
            ENV=$1
            
            if [ -z "$ENV" ]; then
              echo usage: $0 /etc/environment
              exit 1
            fi
            
            grep -c HOSTNAME $ENV || echo HOSTNAME=$HOSTNAME >> $ENV


## New Relic

[Getting started](https://docs.newrelic.com/docs/server/new-relic-servers)

[Release Notes](https://docs.newrelic.com/docs/releases/linux_server/)
