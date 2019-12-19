---
layout: post
title: Spinnaker Secrets
order: 150
---
Storing Spinnaker configs in a git repository is a great solution for maintaining versions of your configurations, but storing secrets in plain text is a bad security practice. As of `halyard-armory:1.4.1`, Spinnaker supports separating your secrets from your configs through end-to-end secrets management. Simply replace secrets in the Halconfig and service profiles with the syntax described here, and Spinnaker will decrypt them as needed. 

{:toc}

## Overview
We can now store secrets (tokens, passwords, sensitive files) separately from the Spinnaker and Halyard configurations as of `halyard-armory:1.4.1`. We'll provide references to these secrets to services that need them.

- Spinnaker services that support decryption will decrypt these secrets upon startup.
- Halyard can decrypt these secrets when it needs to use them (e.g. when validating resources).
- Halyard can send secret references to the services that support decryption or send decrypted secrets if the service does not support it.



## Using Secrets

### Secret Format

When referencing secrets in configs, we use the following general format:

```
encrypted:<secret engine>!<key1>:<value1>!<key2>:<value2>!...
```
The keys and values making up the string vary with each secret engine. Refer to the specific documentation for each engine for more information.

### In Halyard
Halyard can understand the secrets we provided. If the service we're deploying is able to decrypt secrets, Halyard will pass the reference directly, otherwise it will decrypt the configuration before sending it.

For instance, after replacing the github token in our hal config with the encrypted syntax:
```yaml
...
  github:
    enabled: true
    accounts:
    - name: github
      token: encrypted:s3!r:us-west-2!b:mybucket!f:spinnaker-secrets.yml!k:github.token
...
```


We'd find the following in clouddriver.yml:
```yaml
...
  github:
    enabled: true
    accounts:
    - name: github
      token: encrypted:s3!r:us-west-2!b:mybucket!f:spinnaker-secrets.yml!k:github.token
...
```

And for an older release of Clouddriver that does not support decryption:
```yaml
...
  github:
    enabled: true
    accounts:
    - name: github
      token: <TOKEN>
...
```

### Non Halyard configuration
We can also provide secret references directly in `*-local.yml` profile files or directly to Spinnaker services.


### Secret Engines Supported

* [Encrypted S3 buckets](https://docs.armory.io/spinnaker-install-admin-guides/secrets-s3/) (Open Source Spinnaker)
* [Hashicorp Vault](https://docs.armory.io/spinnaker-install-admin-guides/secrets-vault/) (Armory Spinnaker)
* Is there a secret engine you'd like us to support? Submit a feature request [here](http://go.armory.io/support)!