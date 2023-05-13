# zhiva.ai model proxy

Proxy serwer that manages models and uses dicom servers to prepare inference requests.

## Quickstart

> Generate local certificate before start [Use this guide](https://docs.zhiva.ai/latest/setting-up-local-pacs#generate-local-tsl-certificate-only-once-every-365-days)

```shell
docker compose up
```

## servers.json (add DICOMweb server)

path:
`./servers.json`

Keeps all definitions of available pacs servers. It has to be a valid dicomweb server. Each sever has to use uuidv4 which will be used to access it. You can generate such an uuid from [online generator](https://www.uuidgenerator.net/version4).

```javascript
{
  [{uuidv4}]: {
    "uri": string ("https://valid.server.uri/to/dicomweb"),
    "isDefault": boolean (default false),
    "statusUri": string ("https://valid.server.uri/to/status/page"),
    "authType": string ("" | "basicAuth"), 
    "login": string,
    "password": string,
  }
}
```

- `uri` - URI of dicomweb server
- `statusUri` - URI to send request to when status check is needed (has to response with `200` code)
- `isDefault` - boolean which indicates the default server used when request doesn't specify the `uuidv4` value. __One server should have this set to `true`.__
- `statusUri` - (optional)URI to status page. This can be any page accessible through **GET** request that returns stats **200** if server is working correctly. If you don't have any specific page, you can just point to list of studies: `https://valid.server.uri/to/dicomweb/studies`.
- `authType` - Authentication method (`""` | `"basicAuth"`)
- `login` - (optional) Login to use when `authType` is set
- `password` - (optional) Password to use when `authType` is set

## models.json (add model API)

path: `./models.json`

Stores all definitions of inference models. Each model might be available for different type of task. Each model has to use uuidv4 which will be used to access it. You can generate such an uuid from [online generator](https://www.uuidgenerator.net/version4).

```javascript
{
  [{uuidv4}]: {
    "uri": string ("https://valid.model.uri/to/inference"),
    "task": string ("segmentation" | "annotation" | "prediction"),
    "supports": Array<string> (list of supported paths, eg. ["/studies/series"]),
    "statusUri": (optional) string ("https://valid.model.uri/to/status/page"),
    "dataFilter": (optional) Record(DataFilters)
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
- `statusUri` - URI to status page. This can be any page accessible through **GET** request that returns stats **200** if model is working correctly.
- `dataFilter` - (optional) List of [DataFilters](https://github.com/zhiva-ai/model-proxy-server/blob/master/src/types/models.d.ts#L8-L15). Used to filter incoming data.

## Status check

After defining your models and servers you can check if all of them are available through proxy. This is possible by accessing `https://localhost:8002/status` page. This page is only visible when your `docker-compose.yml` configuration has `ZHIVA_SHOW_STATUS_PAGE` variable set to `true`:

```yaml
    environment:
      - ZHIVA_SHOW_STATUS_PAGE=true
```