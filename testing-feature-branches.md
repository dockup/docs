# Using Dockup to test feature branches of multi-container apps

Dockup creates on-demand full stack environments for any git branch
and let you test multiple code changes at the same time.
This enables anyone in the team (or a bot) to quickly get a test environment
which has all your microservices using just a branch name.
This is especially useful for remote teams where people work asynchronously.
Here, we'll see how to create a Dockup Blueprint for a three tier app and use
it to create on-demand environments for testing feature branches.


A Blueprint is the configuration for your stack and these configurations will
be used to create new deployments. It defines all the containers, their environment
variables, ports and commands.

Let's create a blueprint for a simple three-tier app which has a UI, an HTTP API
and a database. Let's start with the database.


<iframe width="560" height="315" src="https://www.youtube.com/embed/G5ys77AABSE?start=6&end=64" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


In the above video, we added a postgres container to our blueprint and used
the following configurations:

1. Container name is "postgres".
2. The docker image is pulled from the official "postgres" repo in docker hub.
3. Port "5432" is private, ie, only accessible by other services inside the deployment.
4. Postgres username and password are set using environment variables.


Next, we'll add the API container to the blueprint.

<iframe width="560" height="315" src="https://www.youtube.com/embed/G5ys77AABSE?start=64&end=130" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In the above video, we added an API container to our blueprint with the following
configurations:

1. Container name is "api".
2. The docker image is built from a linked Github repository named "django-cors".
3. Port "8000" is public. When deployed, this port will be accessible on a public URL.
4. The api container connects to the postgres container. The DB URL is set via an env variable named "DATABASE_URL".
5. The postgres hostname is dynamic and changes for each deployment. So we use a special Dockup variable as the value that gets replaced at runtime.

We can now create a deployment from this Blueprint and test if the API works.
Dockup gives you a deployment form for every blueprint which lets you
create on-demand environments of your services by combining different branches
from different repositories.

<iframe width="560" height="315" src="https://www.youtube.com/embed/G5ys77AABSE?start=130&end=165" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In the above video, we saw that the API container's public port got a unique URL when it got deployed and we used that to test the API.
We'll edit the blueprint and make the api container's port private because it's only needed by the UI.
After that, we'll add a web UI container.

<iframe width="560" height="315" src="https://www.youtube.com/embed/G5ys77AABSE?start=169&end=257" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In the above video, we added a UI container to our blueprint with the following
configurations:

1. Container name is "ui-react".
2. The docker image is built from a linked Github repository named "react-sample".
3. Port "80" is public. When deployed, this port will be accessible on a public URL.
4. The UI container connects to the API container. The API service hostname is set via an env variable named "API_SERVICE".
5. The API hostname is dynamic and changes for each deployment. So we use a special Dockup variable as the value that gets replaced at runtime.

Now we can create a deployment of the entire blueprint and verify that the UI
can fetch data from the API.

<iframe width="560" height="315" src="https://www.youtube.com/embed/G5ys77AABSE?start=260&end=307" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In the above video, we tested the UI and we found a bug. The UI expects a random
value from the API, but it seems to be broken. We'll fix this bug and open a pull
request on Github.

<iframe width="560" height="315" src="https://www.youtube.com/embed/G5ys77AABSE?start=308&end=410" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Dockup listens to push events from Github and when a pull request is opened, a new
deployment is created automatically and it sends the URL for this environment
to your Slack channel.

<iframe width="560" height="315" src="https://www.youtube.com/embed/G5ys77AABSE?start=410" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

This way, nobody in the team waits for QA environments and
multiple code changes get tested in parallel. This also means you get to test
every important feature branch in an isolated environment and pull requests
can be merged with confidence.
