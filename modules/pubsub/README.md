# Google Cloud Pub/Sub Module

This module allows managing a single Pub/Sub topic, including multiple subscriptions and IAM bindings at the topic and subscriptions levels, as well as schemas.

## Examples

### Simple topic with IAM

```hcl
module "pubsub" {
  source     = "./fabric/modules/pubsub"
  project_id = "my-project"
  name       = "my-topic"
  iam = {
    "roles/pubsub.viewer"     = ["group:foo@example.com"]
    "roles/pubsub.subscriber" = ["user:user1@example.com"]
  }
}
# tftest modules=1 resources=3 inventory=simple.yaml
```

### Topic with schema

```hcl
module "topic_with_schema" {
  source     = "./fabric/modules/pubsub"
  project_id = "my-project"
  name       = "my-topic"
  schema = {
    msg_encoding = "JSON"
    schema_type  = "AVRO"
    definition = jsonencode({
      "type" = "record",
      "name" = "Avro",
      "fields" = [{
        "name" = "StringField",
        "type" = "string"
        },
        {
          "name" = "FloatField",
          "type" = "float"
        },
        {
          "name" = "BooleanField",
          "type" = "boolean"
        },
      ]
    })
  }
}
# tftest modules=1 resources=2 inventory=schema.yaml
```

### Subscriptions

Subscriptions are defined with the `subscriptions` variable, allowing optional configuration of per-subscription defaults. Push subscriptions need extra configuration, shown in the following example.

```hcl
module "pubsub" {
  source     = "./fabric/modules/pubsub"
  project_id = "my-project"
  name       = "my-topic"
  subscriptions = {
    test-pull = null
    test-pull-override = {
      labels = { test = "override" }
      options = {
        ack_deadline_seconds       = null
        message_retention_duration = null
        retain_acked_messages      = true
        expiration_policy_ttl      = null
        filter                     = null
      }
    }
  }
}
# tftest modules=1 resources=3 inventory=subscriptions.yaml
```

### Push subscriptions

Push subscriptions need extra configuration in the `push_configs` variable.

```hcl
module "pubsub" {
  source     = "./fabric/modules/pubsub"
  project_id = "my-project"
  name       = "my-topic"
  subscriptions = {
    test-push = null
  }
  push_configs = {
    test-push = {
      endpoint   = "https://example.com/foo"
      attributes = null
      oidc_token = null
    }
  }
}
# tftest modules=1 resources=2
```

### BigQuery subscriptions

BigQuery subscriptions need extra configuration in the `bigquery_subscription_configs` variable.

```hcl
module "pubsub" {
  source     = "./fabric/modules/pubsub"
  project_id = "my-project"
  name       = "my-topic"
  subscriptions = {
    test-bigquery = null
  }
  bigquery_subscription_configs = {
    test-bigquery = {
      table               = "my_project_id:my_dataset.my_table"
      use_topic_schema    = true
      write_metadata      = false
      drop_unknown_fields = true
    }
  }
}
# tftest modules=1 resources=2
```

### Cloud Storage subscriptions

Cloud Storage subscriptions need extra configuration in the `cloud_storage_subscription_configs` variable.

```hcl
module "pubsub" {
  source     = "./fabric/modules/pubsub"
  project_id = "my-project"
  name       = "my-topic"
  subscriptions = {
    test-cloudstorage = null
  }
  cloud_storage_subscription_configs = {
    test-cloudstorage = {
      bucket          = "my-bucket"
      filename_prefix = "test_prefix"
      filename_suffix = "test_suffix"
      max_duration    = "100s"
      max_bytes       = 1000
      avro_config = {
        write_metadata = true
      }
    }
  }
}
# tftest modules=1 resources=2
```
### Subscriptions with IAM

```hcl
module "pubsub" {
  source     = "./fabric/modules/pubsub"
  project_id = "my-project"
  name       = "my-topic"
  subscriptions = {
    test-1 = null
    test-1 = null
  }
  subscription_iam = {
    test-1 = {
      "roles/pubsub.subscriber" = ["user:user1@ludomagno.net"]
    }
  }
}
# tftest modules=1 resources=3
```
<!-- BEGIN TFDOC -->
## Variables

| name | description | type | required | default |
|---|---|:---:|:---:|:---:|
| [name](variables.tf#L94) | PubSub topic name. | <code>string</code> | ✓ |  |
| [project_id](variables.tf#L99) | Project used for resources. | <code>string</code> | ✓ |  |
| [bigquery_subscription_configs](variables.tf#L17) | Configuration parameters for BigQuery subscriptions. | <code title="map&#40;object&#40;&#123;&#10;  table               &#61; string&#10;  use_topic_schema    &#61; bool&#10;  write_metadata      &#61; bool&#10;  drop_unknown_fields &#61; bool&#10;&#125;&#41;&#41;">map&#40;object&#40;&#123;&#8230;&#125;&#41;&#41;</code> |  | <code>&#123;&#125;</code> |
| [cloud_storage_subscription_configs](variables.tf#L28) | Configuration parameters for Cloud Storage subscriptions. | <code title="map&#40;object&#40;&#123;&#10;  bucket          &#61; string&#10;  filename_prefix &#61; string&#10;  filename_suffix &#61; string&#10;  max_duration    &#61; string&#10;  max_bytes       &#61; number&#10;  avro_config &#61; object&#40;&#123;&#10;    write_metadata &#61; bool&#10;  &#125;&#41;&#10;&#125;&#41;&#41;">map&#40;object&#40;&#123;&#8230;&#125;&#41;&#41;</code> |  | <code>&#123;&#125;</code> |
| [dead_letter_configs](variables.tf#L43) | Per-subscription dead letter policy configuration. | <code title="map&#40;object&#40;&#123;&#10;  topic                 &#61; string&#10;  max_delivery_attempts &#61; number&#10;&#125;&#41;&#41;">map&#40;object&#40;&#123;&#8230;&#125;&#41;&#41;</code> |  | <code>&#123;&#125;</code> |
| [defaults](variables.tf#L52) | Subscription defaults for options. | <code title="object&#40;&#123;&#10;  ack_deadline_seconds       &#61; number&#10;  message_retention_duration &#61; string&#10;  retain_acked_messages      &#61; bool&#10;  expiration_policy_ttl      &#61; string&#10;  filter                     &#61; string&#10;&#125;&#41;">object&#40;&#123;&#8230;&#125;&#41;</code> |  | <code title="&#123;&#10;  ack_deadline_seconds       &#61; null&#10;  message_retention_duration &#61; null&#10;  retain_acked_messages      &#61; null&#10;  expiration_policy_ttl      &#61; null&#10;  filter                     &#61; null&#10;&#125;">&#123;&#8230;&#125;</code> |
| [iam](variables.tf#L70) | IAM bindings for topic in {ROLE => [MEMBERS]} format. | <code>map&#40;list&#40;string&#41;&#41;</code> |  | <code>&#123;&#125;</code> |
| [kms_key](variables.tf#L76) | KMS customer managed encryption key. | <code>string</code> |  | <code>null</code> |
| [labels](variables.tf#L82) | Labels. | <code>map&#40;string&#41;</code> |  | <code>&#123;&#125;</code> |
| [message_retention_duration](variables.tf#L88) | Minimum duration to retain a message after it is published to the topic. | <code>string</code> |  | <code>null</code> |
| [push_configs](variables.tf#L104) | Push subscription configurations. | <code title="map&#40;object&#40;&#123;&#10;  attributes &#61; map&#40;string&#41;&#10;  endpoint   &#61; string&#10;  oidc_token &#61; object&#40;&#123;&#10;    audience              &#61; string&#10;    service_account_email &#61; string&#10;  &#125;&#41;&#10;&#125;&#41;&#41;">map&#40;object&#40;&#123;&#8230;&#125;&#41;&#41;</code> |  | <code>&#123;&#125;</code> |
| [regions](variables.tf#L117) | List of regions used to set persistence policy. | <code>list&#40;string&#41;</code> |  | <code>&#91;&#93;</code> |
| [schema](variables.tf#L123) | Topic schema. If set, all messages in this topic should follow this schema. | <code title="object&#40;&#123;&#10;  definition   &#61; string&#10;  msg_encoding &#61; optional&#40;string, &#34;ENCODING_UNSPECIFIED&#34;&#41;&#10;  schema_type  &#61; string&#10;&#125;&#41;">object&#40;&#123;&#8230;&#125;&#41;</code> |  | <code>null</code> |
| [subscription_iam](variables.tf#L133) | IAM bindings for subscriptions in {SUBSCRIPTION => {ROLE => [MEMBERS]}} format. | <code>map&#40;map&#40;list&#40;string&#41;&#41;&#41;</code> |  | <code>&#123;&#125;</code> |
| [subscriptions](variables.tf#L139) | Topic subscriptions. Also define push configs for push subscriptions. If options is set to null subscription defaults will be used. Labels default to topic labels if set to null. | <code title="map&#40;object&#40;&#123;&#10;  labels &#61; map&#40;string&#41;&#10;  options &#61; object&#40;&#123;&#10;    ack_deadline_seconds       &#61; number&#10;    message_retention_duration &#61; string&#10;    retain_acked_messages      &#61; bool&#10;    expiration_policy_ttl      &#61; string&#10;    filter                     &#61; string&#10;  &#125;&#41;&#10;&#125;&#41;&#41;">map&#40;object&#40;&#123;&#8230;&#125;&#41;&#41;</code> |  | <code>&#123;&#125;</code> |

## Outputs

| name | description | sensitive |
|---|---|:---:|
| [id](outputs.tf#L17) | Fully qualified topic id. |  |
| [schema](outputs.tf#L26) | Schema resource. |  |
| [schema_id](outputs.tf#L31) | Schema resource id. |  |
| [subscription_id](outputs.tf#L36) | Subscription ids. |  |
| [subscriptions](outputs.tf#L46) | Subscription resources. |  |
| [topic](outputs.tf#L54) | Topic resource. |  |
<!-- END TFDOC -->
