# Azure Function Core [Tools](https://github.com/Azure/azure-functions-core-tools) Docker image

This Docker image contains the Azure function core tools called `func`

Build the image:

```
docker build -t azurefunc .
```

Then use `func`:

```
docker run --rm -it azurefunc func --help
```
