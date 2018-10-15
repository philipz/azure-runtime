# Running Azure Functions in Knative

This sample is for [JavaScript](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node)
as function language.

Any of Azure's worker runtimes is likely supported.
To add one,
update the build template with Dockerfile content from `func init --docker`.
Add env to switch to port 8080.

## func CLI in docker

The [func](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node) CLI has some dependencies (of which .net wouldn't even install on my old OSX) but it's handy to run docker:

```
docker build -t azure-func func-in-docker/
docker run --rm --name azurefunc -w /workspace -v $(pwd):/workspace -p 7071:7071 -p 8080:8080 -ti azure-func
```

### Func CLI example

The following can run as entrypoint command to [launch](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function-azure-cli#run-the-function-locally) a function from scratch:

```
mkdir /tmp/sample;
cd /tmp/sample;
func init --worker-runtime=node TestRuntime;
func new --name MyHttpTrigger --template "HTTP trigger";
sed -i 's/"authLevel": "function"/"authLevel": "anonymous"/'; MyHttpTrigger/function.json;
func host start --build --port=8080;
```

## Prod build

Test case:

```
docker build -t azure-runtime MyFunctionProj/
docker run --rm --name azure -p 8080:8080 -d azure-runtime
sleep 3
docker logs azure
curl -v http://localhost:8080/api/MyHttpTrigger?name=test1
docker kill azure
```

## Testing with Knative

Please see https://github.com/triggermesh/nodejs-runtime for actual instructions. In brief, with appropriate waits and status checks inbetween:

```
kubectl apply -f build-r00001.yaml

kubectl apply -f route-r00001.yaml

kubectl run -i -t knative-test-client --image=gcr.io/cloud-builders/curl --restart=Never --rm -- \
  --connect-timeout 3 --retry 10 -vSL -w '\n' \
  -H 'Host: azure-runtime-example-function.default.example.com' \
  http://knative-ingressgateway.istio-system.svc.cluster.local/api/MyHttpTrigger?name=Hi%20Knative
```

## Support

We would love your feedback on this project so don't hesitate to let us know what is wrong and how we could improve it, just file an [issue](https://github.com/triggermesh/azure-runtime/issues/new)

## Code of Conduct

This work is by no means part of [CNCF](https://www.cncf.io/) but we abide by its [code of conduct](https://github.com/cncf/foundation/blob/master/code-of-conduct.md)
