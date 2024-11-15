---
# generated by https://github.com/hashicorp/terraform-plugin-docs
page_title: "confluent_schema_registry_dek Resource - terraform-provider-confluent"
subcategory: ""
description: |-
  
---

# confluent_schema_registry_dek Resource

[![General Availability](https://img.shields.io/badge/Lifecycle%20Stage-General%20Availability-%2345c6e8)](https://docs.confluent.io/cloud/current/api.html#section/Versioning/API-Lifecycle-Policy)

`confluent_schema_registry_dek` provides a Schema Registry Data Encryption Key (DEK) resource that enables creating, editing, and deleting Schema Registry Data Encryption Keys on Confluent Cloud.

## Example Usage

### Option #1: Manage multiple Schema Registry clusters in the same Terraform workspace

```terraform
provider "confluent" {
  cloud_api_key    = var.confluent_cloud_api_key    # optionally use CONFLUENT_CLOUD_API_KEY env var
  cloud_api_secret = var.confluent_cloud_api_secret # optionally use CONFLUENT_CLOUD_API_SECRET env var
}

resource "confluent_schema_registry_dek" "my_dek" {
  schema_registry_cluster {
    id = data.confluent_schema_registry_cluster.essentials.id
  }
  rest_endpoint = data.confluent_schema_registry_cluster.essentials.rest_endpoint
  credentials {
    key    = "<Schema Registry API Key for data.confluent_schema_registry_cluster.essentials>"
    secret = "<Schema Registry API Secret for data.confluent_schema_registry_cluster.essentials>"
  }
  
  kek_name = "my_kek"
  subject_name = "my_subject"
  hard_delete = true

  lifecycle {
    prevent_destroy = true
  }
}
```

### Option #2: Manage a single Schema Registry cluster in the same Terraform workspace

```terraform
provider "confluent" {
  schema_registry_id            = var.schema_registry_id            # optionally use SCHEMA_REGISTRY_ID env var
  schema_registry_rest_endpoint = var.schema_registry_rest_endpoint # optionally use SCHEMA_REGISTRY_REST_ENDPOINT env var
  schema_registry_api_key       = var.schema_registry_api_key       # optionally use SCHEMA_REGISTRY_API_KEY env var
  schema_registry_api_secret    = var.schema_registry_api_secret    # optionally use SCHEMA_REGISTRY_API_SECRET env var
}

resource "confluent_schema_registry_dek" "my_dek" {
  kek_name     = "my_kek"
  subject_name = "my_subject"
  hard_delete  = true

  lifecycle {
    prevent_destroy = true
  }
}
```

<!-- schema generated by tfplugindocs -->
## Argument Reference

The following arguments are supported:

- `schema_registry_cluster` - (Optional Configuration Block) supports the following:
  - `id` - (Required String) The ID of the Schema Registry cluster, for example, `lsrc-abc123`.
- `rest_endpoint` - (Optional String) The REST endpoint of the Schema Registry cluster, for example, `https://psrc-00000.us-central1.gcp.confluent.cloud:443`).
- `credentials` (Optional Configuration Block) supports the following:
  - `key` - (Required String) The Schema Registry API Key.
  - `secret` - (Required String, Sensitive) The Schema Registry API Secret.
- `kek_name` - (Required String) The name of the KEK used to encrypt this DEK.
- `subject_name` - (Required String) The subject for this DEK.
- `version` - (Optional Integer) The version of this DEK. Defaults to `1`.
- `algorithm` - (Optional String) Accepted values are: `AES128_GCM`, `AES256_GCM`, and `AES256_SIV`. Defaults to `AES256_GCM`.
- `encrypted_key_material` - (Optional String) The encrypted key material for the DEK.
- `hard_delete` - (Optional Boolean) An optional flag to control whether a DEK should be soft-deleted or hard-deleted. Defaults to `false`.

-> **Note:** A Schema Registry API key consists of a key and a secret. Schema Registry API keys are required to interact with Schema Registry clusters in Confluent Cloud. Each Schema Registry API key is valid for one specific Schema Registry cluster.

-> **Note:** Use Option #2 to simplify the key rotation process. When using Option #1, to rotate a Schema Registry API key, create a new Schema Registry API key, update the `credentials` block in all configuration files to use the new Schema Registry API key, run `terraform apply -target="confluent_schema_registry_dek.pii"`, and remove the old Schema Registry API key. Alternatively, in case the old Schema Registry API Key was deleted already, you might need to run `terraform plan -refresh=false -target="confluent_schema_registry_dek.pii" -out=rotate-schema-registry-api-key` and `terraform apply rotate-schema-registry-api-key` instead.

!> **Warning:** Use Option #2 to avoid exposing sensitive `credentials` value in a state file. When using Option #1, Terraform doesn't encrypt the sensitive `credentials` value of the `confluent_schema_registry_dek` resource, so you must keep your state file secure to avoid exposing it. Refer to the [Terraform documentation](https://www.terraform.io/docs/language/state/sensitive-data.html) to learn more about securing your state file.

## Attributes Reference

In addition to the preceding arguments, the following attributes are exported:

- `id` - (Required String) The ID of the Schema Registry DEK, in the format `<Schema Registry Cluster Id>/<Schema Registry Kek Name>/<Subject>/<Version>/<Algorithm>`, for example, `lsrc-8wrx70/testkek/ts/1/AES256_GCM`.
- `key_material` - (Optional String) The decrypted version of encrypted key material.

## Import
 
You can import a Schema Registry Key by using the Schema Registry cluster ID, KEK name, Subject, Version and Algorithm in the format `<Schema Registry Cluster Id>/<Schema Registry KEK Name>/<Subject>/<Version>/<Algorithm>`, for example:

```shell
$ export IMPORT_SCHEMA_REGISTRY_API_KEY="<schema_registry_api_key>"
$ export IMPORT_SCHEMA_REGISTRY_API_SECRET="<schema_registry_api_secret>"
$ export IMPORT_SCHEMA_REGISTRY_REST_ENDPOINT="<schema_registry_rest_endpoint>"
$ terraform import confluent_schema_registry_dek.my_dek lsrc-8wrx70/testkek/ts/1/AES256_GCM
```

!> **Warning:** Do not forget to delete terminal command history afterwards for security purposes.

## Getting Started
The following end-to-end example might help to get started with field-level encryption:
  * [field-level-encryption-schema](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/field-level-encryption-schema)
