# Running Azure Functions in Knative

Azure functions are one of the serverless offerings available to users. Perhaps unknown to most people is that the runtime of Azure functions is available to build and test functions locally. It has also been [documented](https://medium.com/@asavaritayal/azure-functions-on-kubernetes-75486225dac0) that Azure functions could run in Kubernetes.

This repository is a step by step tutorial to show you how to run functions that use the Azure function runtime in [knative](https://github.com/knative/docs).

This sample is for [JavaScript](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node)
as function language.

Any of Azure's worker runtimes is likely supported.
To add one,
update the build template with Dockerfile content from `func init --docker`.
Add env to switch to port 8080.

## Pre-requisites

For local testing you need:

* [.NET core](https://www.microsoft.com/net/download)
* Azure function core [tools](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)

On OSX for example, once you get the .NET core SDK you can get the Azure function core tools via `brew`:

```
brew tap azure/functions
brew install azure-functions-core-tools 
```

If you struggle to install .NET core and the [Azure functions CLI](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node), you can build it via Docker as shown in the next section.

### Getting the Azure functions `func` CLI via Docker

First build the Docker image using the provided Dockerfile in the `func-in-docker/` directory.

```
docker build -t azure-func func-in-docker/
```

You will then be able to run `func` from an interactive container.

```
docker run --rm --name azurefunc -w /workspace -v $(pwd):/workspace -p 7071:7071 -p 8080:8080 -ti azure-func
func@13a5ece9b783:/workspace$ which func
/usr/bin/func
```

## Local function testing

You are now ready to start developing your function whether directly on your local machine or from within a container.

The following commands shows you the step by step process to [start](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function-azure-cli#run-the-function-locally) a function from scratch:

```
mkdir /tmp/functions
cd /tmp/functions
func init --worker-runtime=node foobar
func new --name foobar --template "HTTP trigger"
sed -i 's/"authLevel": "function"/"authLevel": "anonymous"/' foobar/function.json
func host start --build --port=8080

                  %%%%%%
                 %%%%%%
            @   %%%%%%    @
          @@   %%%%%%      @@
       @@@    %%%%%%%%%%%    @@@
     @@      %%%%%%%%%%        @@
       @@         %%%%       @@
         @@      %%%       @@
           @@    %%      @@
                %%
                %

Azure Functions Core Tools (2.1.725 Commit hash: 68f448fe6a60e1cade88c2004bf6491af7e5f1df)
Function Runtime Version: 2.0.12134.0
[10/19/2018 12:40:27] Building host: startup suppressed:False, configuration suppressed: False
[10/19/2018 12:40:27] Reading host configuration file '/tmp/functions/host.json'
[10/19/2018 12:40:27] Host configuration file read:
...
```

On another terminal do:

```
curl -w '\n\' http://localhost:8080/api/foobar?name=alice
Hello alice
```

If you reach this point you have successfully built a node.js function with the Azure Functions runtime. You can now move on to packaging that function as a container.

## Preparing a Docker image

Test case:

```
docker build -t azure-function MyFunctionProjWithDockerfile/
docker run --rm --name azure -p 8080:8080 -d azure-function
docker logs azure
```

You are then ready to call the function.

```
curl -w '\n' http://localhost:8080/api/MyHttpTrigger?name=alice
```

## Deploying Azure Functions in Knative

Please see the TriggerMesh [nodejs runtime](https://github.com/triggermesh/nodejs-runtime) for complete instructions and explanations.

Knative leverages Kubernetes and Istio to provide automatic scaling and container image build steps. Knative can therefore package your functions in container images and run then with autoscaling turned on.

To build the container images that contain your Azure functions you need to deploy the Build template from this repo:

```
kubectl apply -f knative-build-template.yaml
```

And then as an example, create a Configuration object that will reference this build template and create what are called Knative revisions.

To get traffic to flow to your functions (assuming it is an `HTTP Trigger` type function you can create a Knative route object.

See the provided manifests:

```
kubectl apply -f build-r00001.yaml
kubectl apply -f route-r00001.yaml
```

To call your function you can use the debugging command below which lets your reach the function from within the cluster using the proper Host header.

```
kubectl run -it curl --image=gcr.io/cloud-builders/curl --restart=Never --rm -- \
  --connect-timeout 3 --retry 10 -vSL -w '\n' \
  -H 'Host: azure-runtime-example-function.default.example.com' \
  http://knative-ingressgateway.istio-system.svc.cluster.local/api/foobar?name=Hi%20Knative
```

Enjoy !

## Support

We would love your feedback on this project so don't hesitate to let us know what is wrong and how we could improve it, just file an [issue](https://github.com/triggermesh/azure-runtime/issues/new)

## Code of Conduct

This work is by no means part of [CNCF](https://www.cncf.io/) but we abide by its [code of conduct](https://github.com/cncf/foundation/blob/master/code-of-conduct.md)
