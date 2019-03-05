# GCP agent configuration instructions


### Configuring base domain

After installing agent, you would have configured DNS. You need to specify the
key and domain name without wildcard for agent.

For example, if you own "acme.com" and you want your deployment URLs to be
"<random_string>.dockup.acme.com", then your base domain is dockup.acme.com and
same should be configured in agent settings.

### Configuring registry

After creating project in google, you would have made a note of project id.
Prepend this project id with `gcr.io/` and update registry for agent.

For example, if the id of project created is `dockup-acme`, then the value
of registry will be `gcr.io/dockup-acme`.
