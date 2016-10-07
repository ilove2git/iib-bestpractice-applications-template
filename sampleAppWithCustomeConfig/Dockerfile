FROM dennisseidel/iib-bestpractice-runtimes:10.0.0.6-mqclient

MAINTAINER Dennis Seidel mail@dennisseidel.de

# MANDATORY: BAR Files
# Copy all application bar files into the docker container to /iibProjects/
# from where they are deployed at runtime
COPY app.bar /iibProjects/

# OPTIONALLY: Overwrite Files
# Copy all properties/overwrite files to /iibProperties/. They are applied to
# the bar file with the same name (app.properties applied to app.bar)
# These files can be "overwritten" by putting the properties file into a mountpoint
# or a configmap (kubernetes/openshift specific)
COPY app.properties /iibProperties/

# OPTIONALLY: Custom Config Hook
# Add a file called customconfig.sh with mqsi commands ran at runtime before the
# bar file is deployed.
# This file can be also be stage specific mounted into the image.
COPY customconfig.sh /usr/local/bin/customconfig.sh

# MANDATORY: Passwords
# Password need to be mounted into the location /secret as a env.sh (see example)
# this allow you to set export environment variables for on run when the container start
# those passwords are not visable afterwards.
COPY pw.sh /secret/pw.sh
