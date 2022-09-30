---
layout: default
title: "Local Installation"
description: "Install SCS locally to test extensions or debug the server"
permalink: "/docs/deployment/local-installation.html"
nav_order: 2
grand_parent: Documentation
parent: Deployment
---
# Local Installation

{: .note}
Using a local installation of SCS for production deployments is not
recommended. Please use the Docker image instead

To create a local installation of SCS, for example for debugging or testing
extensions, you can clone the
[simple-configuration-server repository](https://github.com/Tom-Brouwer/simple-configuration-server)
locally. Below is a basic example of using uWSGI to run the application.
You will need:

* A unix operating system (Windows should also work, but not with below steps)
* Python 3.10 needs to be installed (Test using `python3.10 --version`)

Then:
1. Create a .local subdirectory: `mdkir -p .local`
2. Create an app.py file inside the .local directory, with the following
   contents:

   ```python
   import scs

   app = scs.create_app()
   ```
3. Open a terminal (bash) in the root directory of this repository, and do:

   ```bash
   ./install.sh
   source .env/bin/activate
   pip install uwsgi
   export PYTHONPATH=$(pwd)
   # Choose appropriate directories, make sure they exist
   export SCS_CONFIG_DIR=/etc/scs
   export SCS_LOG_DIR=/var/log/scs
   # Now start the app from uWSGI
   cd .local
   uwsgi -s /tmp/scs.sock --manage-script-name --mount /=app:app
   ```

uWSGI should now be running the SCS application. For further documentation, for
example on how to configure NGINX, please look at the [Flask documentation](https://flask.palletsprojects.com/en/2.0.x/deploying/uwsgi/).
