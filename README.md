# Proxmox Backup Server: Client Docker

![Build and Publish Docker Image](https://git.physi.uni-heidelberg.de/tmueller/pbs-client-docker/actions/workflows/build_image.yaml/badge.svg)

## **tl;dr?**

**Q:** What does this Docker image do? 

**A:** It lets you run the Proxmox Backup Server client and backup things from any directory mounted within ``/backups`` inside the container.

## Longer summary

This container is still a work in progress but the main feature (cronjob'd backups) now works. 

Reading and using the ``docker-compose\docker-compose.yml`` and example ``.env.example`` file should be illustrative on how to use this container. 

There are also various helper scripts available when using the shell inside the container 
which can be seen here: https://github.com/Aterfax/pbs-client-docker/tree/main/docker/src/helper_scripts

For more in depth instructions, see: [Using-the-DockerHub-provided-image](#Using-the-DockerHub-provided-image)

## Table of Contents

- [PI Customs](#PI)
- [Quickstart](#Quickstart)
- [Configuration](#Configuration)
- [FAQ](#FAQ)
- [Troubleshooting](#Troubleshooting)
- [Contributing](#Contributing)
- [License](#License)

## PI

### Changes

Encryption can be turned off.
Most of the Data is scientific related and does not contain any secrets and it would be a waste of ressources to encrypt that and loose the deduplication of it because older backups of similar data arent encrypted.
For sensible data you can and should enable the encryption with its own set of keypairs and keep them stored seperate from the backups.

## Quickstart

### Prerequisites

* You are running a platform which supports Docker and you have Docker installed on your system. i.e. a Linux machine or cloud VM. You can download it from [here](https://www.docker.com/get-started).
* You understand how to bind mount / volume mount in docker.


### Using the DockerHub provided image

* Run the image with the provided docker-compose file after amending it and the ``.env`` file where needed.
  * If allowing the container to conduct an auto setup, don't set a  ``PBS_ENCRYPTION_PASSWORD`` value yet as the container first run will autogenerate one for you.
  * Supply your desired ``master-public.pem``, ``master-private.pem`` and ``encryption-key.json`` files with a matching ``PBS_ENCRYPTION_PASSWORD`` or allow the container to automatically generate these for you on first run.
  * The required permissions in your PBS server instance for an API key will be "DatastoreBackup" and "DatastoreReader" on the appropriate path. Or you can supply a username and password combination which has these permissions - this is not recommended!
  * bind mount the folders or volumes you wish to backup into the container's ``/backups`` directiory.
* **If you do allow automatic generation, passwords will be echoed to the container logs only once during first run!** Ensure you do not restart the container before saving these values.
* Backup the ``master-public.pem``, ``master-private.pem`` and ``encryption-key.json`` files and passwords to a separate storage system accessible even if the machine / system being backed up or restored becomes inaccessible.
* Populate the environment file with the correct ``PBS_ENCRYPTION_PASSWORD`` value from the container logs or from your provided ``pem`` /  ``encryption-key.json`` files.
* Restart the container and check the logs to confirm a successful backup.
  * You can start a backup with the ``backupnow`` command from inside the container, i.e. ``docker exec -it pbs-client backupnow``
* Review the other helper scripts [from here](https://github.com/Aterfax/pbs-client-docker/tree/main/docker/src/helper_scripts) which are also available from the container terminal.


### Using a self built image

* Navigate to the ``docker`` subdirectory and run the command ``docker build -t $USER/pbs-client .``
* Take a note of the final line of this command's output e.g. :

        => => naming to docker.io/myusername/pbs-client

* Amend the [docker-compose/docker-compose.yml](docker-compose/docker-compose.yml) image line to: 
  
        image: myusername/pbs-client

* You can now follow the instructions as per the [Using the DockerHub provided image](#Using-the-DockerHub-provided-image) section.


## Configuration

### Common post installation configuration actions

To be filled in.

### Table of available container environment variables

To be filled in.

The following environment variables can be configured to customize the behavior of the Docker container:

| Variable Name      | Default Docker Compose Value | Valid Values           | Description                                                                                                           |
|--------------------|------------------------------|------------------------|-----------------------------------------------------------------------------------------------------------------------|
| PBS_ENDPOINT       | none                         | fqdn                   | target PBS-server                                                                                                     |
| PBS_FINGERPRINT    | none                         | XX:XX:XX:XX...         | your fringerprint of your pbs instance                                                                                |
| PBS_DATASTORE      | none                         | string                 | name of your pbs datastore                                                                                            |
| PBS_DATASTORE_ns   | none                         | string                 | name of your pbs namespace                                                                                            |
| CRON_SCHEDULE      | none                         | * * * * *              | cron expression to define the shedule for backups                                                                     |
| CRON_BACKUP_ONLY   | none                         | boolean                | controls if first backup will be done at the first start                                                              |
| PBS_API_KEY_NAME or PBS_USER | none               | string                 | credentials for pbs either define an api token or specify user                                                        | 
| PBS_API_KEY_SECRET or PBS_PASSWORD | none         | string                 | password or secret                                                                                                    |
| TZ                 | none                         | IANA's time zone database long | Timezone to use for tuimestamps in backup                                                                     |
| UNENCRYPTED        | 0                            | boolean                | disables encryption if set to 1                                                                                       |
| CHANGE_DETECT_MODE | metadata                     | metadata, data, legacy | Let you choose the desired mode for detecting file changes between backups                                            |

## FAQ

### Error: Function not implemented (os error 38)

It appears that some Synology NAS platforms are missing support for the ``getrandom`` system call used during key creation which will block the container from starting. 

To bypass this issue you can manually create some keys or run the container on another platform to generate keys with a random password then transfer these (``encryption-key.json``  ``master-private.pem``  ``master-public.pem``) to the Synology docker container's ``/root/.config/proxmox-backup/`` bind mount.

See also:
- https://github.com/Aterfax/pbs-client-docker/issues/8
- https://forum.proxmox.com/threads/backup-client-encryption-not-working-inside-docker-container.139054/

## Troubleshooting

If you encounter issues, check the [Troubleshooting section](TROUBLESHOOTING.md)  for solutions to common problems.

If this section is lacking steps to resolve your issue please take a look in the Github discussions to see if someone else has already resolved your issue or 
please start a thread.

If you have a problem or feature request and you know this related directly to the code implemented by this repo please file an issue detailing the nature of the problem or feature and any steps for implementation within a pull request.

## Contributing

If you'd like to contribute to this project, follow these steps:

* Fork the repository.
* Create a new branch for your feature: git checkout -b feature-name.
* Make your changes and commit them e.g. : git commit -m "Add feature".
* Push to the branch: git push origin feature-name.
* Create a pull request explaining your changes.

## License

This project is licensed under the [GNU General Public License v3 (GPL-3)](https://www.tldrlegal.com/license/gnu-general-public-license-v3-gpl-3).

In short: You may copy, distribute and modify the software as long as you track changes/dates in source files. Any modifications to or software including (via compiler) GPL-licensed code must also be made available under the GPL along with build & install instructions.
