# Short Note:

- The pipeline initially failed at the `FROM node:14-highspeed` line because the image does not exist.

- I updated the Docker base image to `node:14-alpine` to allow the build to proceed.

- After this change, the pipeline failed again due to outdated and vulnerable dependencies detected by the security scan.

- I then changed the base image to `node:20-alpine` because `node:14-alpine` is too outdated and maybe no longer well supported.

- Fix: I replaced `RUN npm install` with `RUN npm install -g npm@latest` to upgrade npm and pull in patched dependency versions.