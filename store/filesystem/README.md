# FileSystem Catalog

The FileSystem catalog extension allows the registry to pull static service information from a file in addition to supporting standard registration via REST API.
This catalog is useful when you have services that do not run the Amalgam8 protocol (e.g., database service), but you want other services to discover these services using the standard Amalgam8 discovery REST API.

The name of the config file should be `<namespace>.conf` and the format of the file is a JSON object defined by the following schema: 

```
{
  "type": "object",
  "properties": {
    "instances": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "service_name": {
            "type": "string"
          },
          "endpoint": {
            "type": "object",
            "properties": {
              "type": {
                "type": "string"
              },
              "value": {
                "type": "string"
              }
            },
            "required": [
              "type",
              "value"
            ]
          },
          "ttl": {
            "type": "integer",
            "default": 30
          },
          "status": {
            "type": "string",
            "enum": ["STARTING", "UP", "OUT_OF_SERVICE"],
            "default": "UP"
          },
          "tags": {
            "type": "array",
            "items": {
              "type": "string"
            }
          },
          "metadata": {
            "type": "object"
          }
        },
        "required": [
          "service_name",
          "endpoint"
        ]
      }
    }
  },
  "required": [
    "instances"
  ]
}
```

See [example.conf](example.conf) for an example config file.
