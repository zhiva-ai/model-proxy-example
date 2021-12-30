# zhiva.ai model proxy

Proxy serwer that manages models and uses dicom servers to prepare inference requests.

## Quickstart

> Generate local certificate before start [Use this guide](https://docs.zhiva.ai/latest/setting-up-local-pacs#generate-local-tsl-certificate-only-once-every-365-days)

```shell
docker-compose up
```

## servers.json (add DICOMweb server)

path:
`./servers.json`

Keeps all definitions of available pacs servers. It has to be a valid dicomweb server. Each sever has to use uuidv4 which will be used to access it. You can generate such an uuid from [online generator](https://www.uuidgenerator.net/version4).

```javascript
{
  [{uuidv4}]: {
    "uri": string ("https://valid.server.uri/to/dicomweb"),
    "isDefault": boolean (default false)
  }
}
```

- `uri` - URI of dicomweb server
- `isDefault` - boolean which indicates the default server used when request doesn't specify the `uuidv4` value. __One server should have this set to `true`.__

## models.json (add model API)

path: `./models.json`

Stores all definitions of inference models. Each model might be available for different type of task. Each model has to use uuidv4 which will be used to access it. You can generate such an uuid from [online generator](https://www.uuidgenerator.net/version4).

```javascript
{
  [{uuidv4}]: {
    "uri": string ("https://valid.model.uri/to/inference"),
    "task": string ("segmentation" | "annotation" | "prediction"),
    "supports": Array<string> (list of supported paths, eg. ["/studies/series"])
  }
}
```

- `uri` - URI of the inference model API (eg. `http://localhost:8011/predict`)
- `task` - Task which model performs (one of `segmentation` | `annotation` | `prediction`)
- `supports` - List of available paths that model supports:
```shell
/studies
/studies/series
/studies/series/instances
```
