# dtsse-shared-infrastructure

Shared infrastructure pipeline for the DTS Software Engineering team.

- appinsights
- key vault (`dtsse-<env>`)
- managed Grafana dashboard
- postgres DB for the dashboard data

## Key vault access

The vault uses **access policies**, not RBAC. Two of them come from
[`cnp-module-key-vault`](https://github.com/hmcts/cnp-module-key-vault):

| Group | Secrets | Set by |
| --- | --- | --- |
| `DTS CFT Developers` | Get, List | the module's `developers_group` default, in `developer-access.tf` |
| `DTS CFT Software Engineering` | List, Set, Delete, Recover | `product_group_name` in `main.tf`, via `team-access.tf` |

Writing a secret by hand therefore needs membership of **DTS CFT Software Engineering**; Developers is
read-only.

If `az keyvault secret set` returns `Forbidden` for somebody in that group, check the policy is actually on the
vault — it has been missing, and this pipeline recreates it:

```bash
az keyvault show --name dtsse-aat \
  --query "properties.accessPolicies[?objectId=='$(az ad group show --group 'DTS CFT Software Engineering' --query id -o tsv)']"
```

An empty result means the policy has drifted away and the pipeline needs re-running. An empty `terraform plan`
beside an empty result means the state is stale rather than the vault, which needs a refresh first.

## Documentation

August 2024 - Created new TF variables for Major version, API Key Enabled and Zone Redundancy for upgrade to version 10

Here are some useful references:

* https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/dashboard_grafana#grafana_major_version
* https://azure.microsoft.com/en-gb/updates/action-recommended-update-to-using-grafana-version-10-for-azure-managed-grafana/
* https://azure.microsoft.com/en-gb/products/managed-grafana
* https://community.grafana.com/
* https://grafana.com/docs/grafana/latest/whatsnew/