# **10. Hashicorp Vault**

[Embedded exporter](https://github.com/hashicorp/vault/blob/master/website/content/docs/configuration/telemetry.mdx#prometheus)


### **1. Vault sealed**

**Vault instance is sealed on `{{ $labels.instance }}`**

```
  - alert: VaultSealed
    expr: vault_core_unsealed == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Vault sealed (instance {{ $labels.instance }})
      description: "Vault instance is sealed on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **2. Vault too many pending tokens**

**Too many pending tokens `{{ $labels.instance }}: {{ $value | printf "%.2f"}}%`**

```
  - alert: VaultTooManyPendingTokens
    expr: avg(vault_token_create_count - vault_token_store_count) > 0
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Vault too many pending tokens (instance {{ $labels.instance }})
      description: "Too many pending tokens {{ $labels.instance }}: {{ $value | printf \"%.2f\"}}%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

### **3. Vault too many infinity tokens**

**Too many infinity tokens`{{ $labels.instance }}: {{ $value | printf "%.2f"}}%`**

```
  - alert: VaultTooManyInfinityTokens
    expr: vault_token_count_by_ttl{creation_ttl="+Inf"} > 3
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Vault too many infinity tokens (instance {{ $labels.instance }})
      description: "Too many infinity tokens {{ $labels.instance }}: {{ $value | printf \"%.2f\"}}%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


