# Secret Stores
Generates ExternalSecret and SecretStore resources based on definitions
provided from upstream Helm charts.

## Usage
Example values file from an upstream Helm chart calling this chart:
```yaml
secretstores:
  stores:
    aws:
      type: "aws"
      iamRole: "arn:aws:iam::XXXXXXXXXXXX:role/my-service-role"
      region: "us-west-2"
      secrets:
        - sourcePath: "test/my-service/test-secret"
          sourceKey: "nunya"
          destName: "test-secret-aws"
          destKey: "nun-ya"
    vault-service:
      type: "vault"
      server: "http://cool.vault.path:port"
      path: "service"
      secrets:
        - sourcePath: "my-service/test-secret"
          sourceKey: "nunya"
          destName: "test-secret-vault-service"
          destKey: "nun-ya"
    vault-service-shared:
      type: "vault"
      server: "http://cool.vault.path:port"
      path: "service-shared"
      secrets:
        - sourcePath: "shared-secret"
          sourceKey: "everyoneknows"
          destName: "letthemeat"
          destKey: "cake"
```

## Caveats
* `secrets.destName` must be unique across all stores and secrets in the
namespace.
* `secrets.refreshInterval` can be optionally specified, however it is
defaulted to 1 hour.
