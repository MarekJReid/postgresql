This is the databse service for the SUS service 

Once this has been initialised on the local vm w need to include steps to "clone" the current db contents (on the vm) and apply it to the local instance.
note: env variable process still to be setup!


docker compose file 

version: '3'

services:
  postgres:
    image: postgres:latest
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: pw
      POSTGRES_USER: postgres

  db_init:
    build:
      context: .
      dockerfile: DbInitDockerfile
    depends_on:
      - postgres

volumes:
  db_data:

db init file:
FROM debian:bullseye-slim

# Install PostgreSQL client and SSH client
RUN apt-get update && apt-get install -y postgresql-client openssh-client && apt-get clean

# Copy the fetch and restore script
COPY fetch_and_restore.sh /fetch_and_restore.sh
RUN chmod +x /fetch_and_restore.sh

# Run the script
CMD ["/fetch_and_restore.sh"]


fetch and restore bash script

#!/bin/bash

# Fetch the dump from VM
scp user@your_vm_ip:/path/to/dump.sql /tmp/dump.sql

# Wait for PostgreSQL to initialize
until PGPASSWORD=your_password psql -h postgres -U postgres -c '\q'; do
  sleep 1
done

# Restore the dump
PGPASSWORD=your_password psql -h postgres -U postgres your_local_db_name < /tmp/dump.sql


Instructions:

Place the docker-compose.yml, DbInitDockerfile, and fetch_and_restore.sh in the root directory.
Ensure to set up SSH key-based authentication to the VM so that scp doesn't prompt for a password.
Run docker-compose up. This will start both the PostgreSQL container and the db_init service. The db_init service will fetch the dump from the VM and restore it to the PostgreSQL container.
