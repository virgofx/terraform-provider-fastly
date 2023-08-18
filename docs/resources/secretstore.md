---
layout: "fastly"
page_title: "Fastly: secretstore"
sidebar_current: "docs-fastly-resource-secretstore"
description: |-
  A secret store is a persistent, globally distributed store for secrets accessible to Compute@Edge services during request processing.
---

# fastly_secretstore

A secret store is a persistent, globally distributed store for secrets accessible to Compute@Edge services during request processing.

In order for a Secret Store (`fastly_secretstore`) to be accessible to a [Compute@Edge](https://developer.fastly.com/learning/compute/) service you'll first need to define a Compute service (`fastly_service_compute`) in your configuration, and then create a link to the Secret Store from within the service using the `resource_link` block (shown in the below examples).

~> **Warning:** Unlike other stores (Config Store, KV Store etc) deleting a Secret Store will automatically delete all the secrets it contains. There is no need to manually delete the secrets first.

~> **Note:** The Fastly Terraform provider does not provide a means to seed the Secret Store with secrets (this is because the values are persisted into the Terraform state file as plaintext). To populate the Secret Store with secrets please use the [Fastly API](https://developer.fastly.com/reference/api/services/resources/secret-store-secret/) directly or the [Fastly CLI](https://developer.fastly.com/reference/cli/secret-store-entry/).

## Example Usage

Basic usage:

```terraform
# IMPORTANT: Deleting a Secret Store requires first deleting its resource_link.
# This requires a two-step `terraform apply` as we can't guarantee deletion order.
# e.g. resource_link deletion within fastly_service_compute might not finish first.
resource "fastly_secretstore" "example" {
  name = "my_secret_store"
}

resource "fastly_service_compute" "example" {
  name = "my_compute_service"

  domain {
    name = "demo.example.com"
  }

  package {
    filename         = "package.tar.gz"
    source_code_hash = data.fastly_package_hash.example.hash
  }

  resource_link {
    name        = "my_resource_link"
    resource_id = fastly_secretstore.example.id
  }

  force_destroy = true
}

data "fastly_package_hash" "example" {
  filename = "package.tar.gz"
}
```

## Import

Fastly Secret Stores can be imported using their Store ID, e.g.

```sh
$ terraform import fastly_secretstore.example xxxxxxxxxxxxxxxxxxxx
```

<!-- schema generated by tfplugindocs -->
## Schema

### Required

- `name` (String) A human-readable name for the Secret Store. The value must contain only letters, numbers, dashes (-), underscores (_), or periods (.). It is important to note that changing this attribute will delete and recreate the Secret Store, and discard the current entries. You MUST first delete the associated resource_link block from your service before modifying this field.

### Read-Only

- `id` (String) The ID of this resource.