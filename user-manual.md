# User manual

This guide will help you get started with Dockup.

## Step 1. Configure your container source

### If you already use a docker image registry
If you already build and push Docker images of your services somewhere,
Dockup can pull these images to start your containers. For this, you
have to provide the image pull secrets if the images are private - for example,
on GCR in GCP or ECR in AWS. You go to "Settings > Image Pull Secrets" to do this.

Here's an example to show how you can create an image pull secret in Dockup
to fetch images from GCR:

![GCR image pull secret](/images/gcr_pull_secret.png)

Explanation of fields:

1. Name - This is just a name you will use later to identify this pull secret.
2. Server - This is the docker registry server. Can be found in ~/.docker/config.
3. Username - The username used for registry authentication.
4. Passowrd - The password used for registry authentication.

Need help with a specific registry? Contact support@getdockup.

### If you don't use an image registry
If you don't build images yourself, Dockup can do it for you if you link your
Github repos. Go to "Settings > Install Github App" to do this. Dockup assumes
you have a "Dockerfile" in the root directory of your github repo.

## Step 2. Create a Blueprint
A blueprint is the configuration for all the containers you'll be running for
each deployment. This step will be easy if you already
know the environment variables to run your containers and the ports they listen
on.

Go to "Blueprint > Create another blueprint"

TODO
