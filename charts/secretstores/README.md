# Secret Stores
Generates ExternalSecret and SecretStore resources based on definitions
provided from upstream Helm charts.

## Usage
Secret stores can be defined in a variety of ways. The most common way is to
allow the store to discover and import all secrets to which it has access.
```yaml
secretstores:
  stores:
    # AWS example
    aws:
      type: "aws"
      iamRole: "arn:aws:iam::XXXXXXXXXXXX:role/my-service-role"
      region: "us-west-2"
    # Vault example for service/ mount
    vault-service:
      type: "vault"
      server: "http://cool.vault.path:port"
      path: "service"
    # Vault example for service-shared/ mount
    vault-service-shared:
      type: "vault"
      server: "http://cool.vault.path:port"
      path: "service-shared"
```

This will create a single Kubernetes secret per store (so three in this
example) with all secrets injected into it.  The names of the secrets
follow the pattern `secrets-[store]`, and contain keys corresponding to the
imported external secrets.  For example, the `secrets-aws` secret may look
like this:
```yaml
test_shared_test-secret: '{"another":"thing"}'
test_some-testserver_test-secret: '{"nunya":"bizniz"}'
```

Tracing the origin secrets back to AWS Secrets Manager, this means that the
following secrets would exist:
* `/test/shared/test-secret`: `{"another":"thing"}`
* `/test/some-testserver/test-secret`: `{"nunya":"bizniz"}`

Another method of defining secrets is to specify them directly in the store
under the `secrets` key.  This is useful if you want to explicitly grab only
individual secrets.
```yaml
secretstores:
  stores:
    # AWS example
    aws:
      type: "aws"
      iamRole: "arn:aws:iam::XXXXXXXXXXXX:role/my-service-role"
      region: "us-west-2"
      secrets:
        - sourcePath: "test/my-service/test-secret"
          sourceKey: "nunya"
        - sourcePath: "test/my-service/another-test-secret"
          sourceKey: "nunya"
          destKey: "nunya2"
    # Vault example for service/ mount
    vault-service:
      type: "vault"
      server: "http://cool.vault.path:port"
      path: "service"
      secrets:
        - sourcePath: "my-service/test-secret"
          sourceKey: "nunya"
    # Vault example for service-shared/ mount
    vault-service-shared:
      type: "vault"
      server: "http://cool.vault.path:port"
      path: "service-shared"
      secrets:
        - sourcePath: "shared-secret"
          sourceKey: "everyoneknows"
```

The final option is to include `additionalSecretsPaths` in the store
definitions.  As long as the store's role has access to the paths,
they will be included in the resultant secret.
```yaml
secretstores:
  stores:
    # AWS example
    aws:
      type: "aws"
      iamRole: "arn:aws:iam::XXXXXXXXXXXX:role/my-service-role"
      region: "us-west-2"
      additionalSecretsPaths:
        - "test/sibling-service/another-secret"
    # Vault example for service/ mount
    vault-service:
      type: "vault"
      server: "http://cool.vault.path:port"
      path: "service"
      additionalSecretsPaths:
        - "sibling-service/secret"
    # Vault example for service-shared/ mount
    vault-service-shared:
      type: "vault"
      server: "http://cool.vault.path:port"
      path: "service-shared"
      additionalSecretsPaths:
        - "nested/shared-secret"
```

## Caveats
* For `aws` type stores, the default is to pull from three locations:
   * `[env]/[service-name]`
   * `[env]/shared`
   * `shared`
* For `vault` type stores, the default is to only pull from one location in the
mount:
   * `[service-name]`
* `secretstores.stores.[store].secrets.destKey` is optional, and only required
if there are other key collisions between source secrets.
* `secretstores.stores.[store].refreshInterval` can be optionally specified,
however it is defaulted to 1 hour.
