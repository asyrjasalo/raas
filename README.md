Robots as a Service
===================

## Usage

Robot Framework and [RESTinstance](https://github.com/asyrjasalo/RESTinstance) are currently provided:

    curl https://funk.raas.dev/function/robot \
      --header "apikey: {{password}}" \
      --data-binary @functions/robotwrapper/rest.robot

You can ran tasks in background (async) and get back results when done:

    curl https://funk.raas.dev/async-function/robot \
      --header "apikey: {{password}}" \
      --header "X-Callback-Url: https://funk.raas.dev/function/robot" \
      --data-binary @functions/robotwrapper/callbacked.robot

The URL in header `X-Callback-Url` is called after the execution has finished.



## Development

### Environments

- Functions are best developed in local Docker Swarm - fastest to debug here
- Infrastructure and services are best developed in local k8s (kind)
- Google Kubernetes Engine is cheapest in production

### Tech

- Kubernetes (GKE/AKS) and Helm for deployment
- OpenFaaS-cloud, for running microservices and functions
- Kong API Gateway, for authentication and authorization
- Node-RED, for graphical editor for automation flows
- Robot Framework, for running tasks

### Requirements

Homebrew casks work on OS X:

    brew cask install docker-ce
    brew cask install google-cloud-sdk

On Homebrew (OS X) or Linuxbrew:

    brew install faas-cli
    brew install kubernetes-cli
    brew install go

### Workflow

1. `setup_` - creates the base infrastructure
2. `build_` - upgrades services and APIs
3. `deploy_` - deploys to OpenFaaS
4. `test_` - tests over HTTPS(S)



## Kubernetes secrets

### registry-auth

    cd secrets
    htpasswd -Bbn faas "1^p?W>x5oiWh" > registry-auth

Put `registry-auth` content in the chart's `values.yaml` property `htpasswd`.

### robotred-ingress-auth

    cd secrets
    htpasswd -bn earlyrobot "WsB0q7-1RlKv" > auth

    kubectl create secret generic robotred-ingress-auth --from-file=auth --namespace robotred

### kong-admin-ingress-auth

    cd secrets
    htpasswd -bn admin "w*5ZCUuFpyQn4bp-" > auth

    kubectl create secret generic kong-admin-ingress-auth --from-file=auth --namespace kong



## Kong administration

### Development

Accessible in [:8001](http://localhost:8001) for debugging purposes.

Running `kong/dev/1_dev_kongadmins` creates a loopback for accessing
Kong Admin API via Kong.

After running the script , prefer Kong endpoint
[http://localhost:8000/fadmin](http://localhost:8000/fadmin) for
authenticating to Kong Admin API, as it works this way in production.

This is also how the rest of the numbered scripts in `kong/dev/*` access
the Kong Admin API.

Konga can be used as a web admin GUI, running on [:1337](http://localhost:1337).
You can use `http://kong:8001` as the URL for connecting to Kong Admin API directly, or prefer key authentication, with endpoint `http://kong:8000/fadmin`
to connect to it via the created loopback.

### Production

Create Kong admin ingress:

    kubectl apply -f kubernetes/kong-admin-ingress.yaml --namespace kong

Run `kong/prod/1_prod_fadmin` to create the loopback for Kong Admin API.
From then, use Kong consumer `fadmin` for proxying from Kong to Kong Admin API,
similarly as done in development environment.

Disable admin ingress for security after you have ran `kong/prod/1_prod_fadmin`:

    kubectl delete -f kubernetes/kong-admin-ingress.yaml --namespace kong

Then run rest of the `kong/prod/` scripts to create the rest of the services.
