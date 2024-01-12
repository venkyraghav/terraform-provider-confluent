---
# generated by https://github.com/hashicorp/terraform-plugin-docs
page_title: "confluent_flink_statement Resource - terraform-provider-confluent"
subcategory: ""
description: |-
  
---

# confluent_flink_statement Resource

[![Preview](https://img.shields.io/badge/Lifecycle%20Stage-Preview-%2300afba)](https://docs.confluent.io/cloud/current/api.html#section/Versioning/API-Lifecycle-Policy)

-> **Note:** `confluent_flink_statement` resource is available in **Preview** for early adopters. Preview features are introduced to gather customer feedback. This feature should be used only for evaluation and non-production testing purposes or to provide feedback to Confluent, particularly as it becomes more widely available in follow-on editions.  
**Preview** features are intended for evaluation use in development and testing environments only, and not for production use. The warranty, SLA, and Support Services provisions of your agreement with Confluent do not apply to Preview features. Preview features are considered to be a Proof of Concept as defined in the Confluent Cloud Terms of Service. Confluent may discontinue providing Preview releases of the Preview features at any time in Confluent’s sole discretion.

-> **Note:** It is recommended to set `lifecycle { prevent_destroy = true }` on production instances to prevent accidental statement deletion. This setting rejects plans that would destroy or recreate the statement, such as attempting to change uneditable attributes. Read more about it in the [Terraform docs](https://www.terraform.io/language/meta-arguments/lifecycle#prevent_destroy).

## Example Usage

### Option #1: Manage multiple Flink Compute Pools in the same Terraform workspace

```terraform
provider "confluent" {
  cloud_api_key    = var.confluent_cloud_api_key    # optionally use CONFLUENT_CLOUD_API_KEY env var
  cloud_api_secret = var.confluent_cloud_api_secret # optionally use CONFLUENT_CLOUD_API_SECRET env var
}

resource "confluent_flink_statement" "random_int_table" {
  compute_pool {
    id = confluent_flink_compute_pool.example.id
  }
  principal {
    id = confluent_service_account.app-manager-flink.id
  }
  statement  = "CREATE TABLE random_int_table(ts TIMESTAMP_LTZ(3), random_value INT);"
  properties = {
    "sql.current-catalog"  = data.confluent_environment.example.display_name
    "sql.current-database" = data.confluent_kafka_cluster.example.display_name
  }
  rest_endpoint   = data.confluent_flink_compute_pool.example.rest_endpoint
  credentials {
    key    = confluent_api_key.env-admin-flink-api-key.id
    secret = confluent_api_key.env-admin-flink-api-key.secret
  }

  lifecycle {
    prevent_destroy = true
  }
}
```

### Option #2: Manage a single Flink Compute Pool in the same Terraform workspace

```terraform
provider "confluent" {
  flink_compute_pool_id = var.flink_compute_pool_id      # optionally use FLINK_COMPUTE_POOL_ID env var
  flink_rest_endpoint   = var.flink_rest_endpoint        # optionally use FLINK_REST_ENDPOINT env var
  flink_api_key         = var.flink_api_key              # optionally use FLINK_API_KEY env var
  flink_api_secret      = var.flink_api_secret           # optionally use FLINK_API_SECRET env var
  flink_principal_id    = var.flink_principal_id         # optionally use FLINK_PRINCIPAL_ID env var
}

resource "confluent_flink_statement" "example" {
  statement  = "CREATE TABLE random_int_table(ts TIMESTAMP_LTZ(3), random_value INT);"
  properties = {
    "sql.current-catalog"  = var.confluent_environment_display_name
    "sql.current-database" = var.confluent_kafka_cluster_display_name
  }

  lifecycle {
    prevent_destroy = true
  }
}
```

<!-- schema generated by tfplugindocs -->
## Argument Reference

The following arguments are supported:

- `compute_pool` - (Optional Configuration Block) supports the following:
    - `id` - (Required String) The ID of the Flink Compute Pool, for example, `lfcp-abc123`.
- `principal` - (Optional Configuration Block) supports the following:
    - `id` - (Required String) The ID of the Principal the Flink Statement runs as, for example, `sa-abc123`.
- `statement` - (Required String) The raw SQL text statement, for example, `SELECT CURRENT_TIMESTAMP;`.
- `statement_name` - (Optional String) The ID of the Flink Statement, for example, `cfeab4fe-b62c-49bd-9e99-51cc98c77a67`.
- `rest_endpoint` - (Optional String) The REST endpoint of the Flink Compute Pool, for example, `https://flink.us-east-1.aws.stag.cpdev.cloud/sql/v1beta1/organizations/1111aaaa-11aa-11aa-11aa-111111aaaaaa/environments/env-abc123`).
- `credentials` (Optional Configuration Block) supports the following:
    - `key` - (Required String) The Flink API Key.
    - `secret` - (Required String, Sensitive) The Flink API Secret.

-> **Note:** A Flink API key consists of a key and a secret. Flink API keys are required to interact with Flink Statements in Confluent Cloud. Each Flink API key is valid for one specific Flink Region.

-> **Note:** Use Option #2 to simplify the key rotation process. When using Option #1, to rotate a Flink API key, create a new Flink API key, update the `credentials` block in all configuration files to use the new Flink API key, run `terraform apply -target="confluent_flink_statement.example"`, and remove the old Flink API key. Alternatively, in case the old Flink API Key was deleted already, you might need to run `terraform plan -refresh=false -target="confluent_flink_statement.example" -out=rotate-flink-api-key` and `terraform apply rotate-flink-api-key` instead.

- `properties` - (Optional Map) The custom topic settings to set:
    - `name` - (Required String) The setting name, for example, `sql.local-time-zone`.
    - `value` - (Required String) The setting value, for example, `GMT-08:00`.

- `stopped` - (Optional Boolean) The boolean flag to control whether the running Flink Statement should be stopped. Defaults to `false`.

!> **Warning:** Use Option #2 to avoid exposing sensitive `credentials` value in a state file. When using Option #1, Terraform doesn't encrypt the sensitive `credentials` value of the `confluent_flink_statement` resource, so you must keep your state file secure to avoid exposing it. Refer to the [Terraform documentation](https://www.terraform.io/docs/language/state/sensitive-data.html) to learn more about securing your state file.

## Attributes Reference

In addition to the preceding arguments, the following attributes are exported:

- `id` - (Required String) The ID of the Flink statement, in the format `<Environment ID>/<Flink Compute Pool ID>/<Flink Statement name>`, for example, `env-abc123/lfcp-xyz123/cfeab4fe-b62c-49bd-9e99-51cc98c77a67`.
- `resource_version` - (Required String) The ID of the Flink statement's version, for example, `2`.

## Import

You can import a Flink topic by using the Flink Statement name, for example:

```shell
# Option #1: Manage multiple Flink Compute Pools in the same Terraform workspace
$ export IMPORT_FLINK_COMPUTE_POOL_ID="<flink_compute_pool_id>"
$ export IMPORT_FLINK_API_KEY="<flink_api_key>"
$ export IMPORT_FLINK_API_SECRET="<flink_api_secret>"
$ export IMPORT_FLINK_REST_ENDPOINT="<flink_rest_endpoint>"
$ export IMPORT_FLINK_PRINCIPAL_ID="<flink_rest_endpoint>"
$ terraform import confluent_flink_statement.example cfeab4fe-b62c-49bd-9e99-51cc98c77a67

# Option #2: Manage a single Flink Compute Pool in the same Terraform workspace
$ terraform import confluent_flink_statement.example cfeab4fe-b62c-49bd-9e99-51cc98c77a67
```

!> **Warning:** Do not forget to delete terminal command history afterwards for security purposes.

## Getting Started
The following end-to-end example might help to get started with [Flink Statements](https://docs.confluent.io/cloud/current/flink/get-started/overview.html):
  * [flink-quickstart](https://github.com/confluentinc/terraform-provider-confluent/tree/master/examples/configurations/flink-quickstart)