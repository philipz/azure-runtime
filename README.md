
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
