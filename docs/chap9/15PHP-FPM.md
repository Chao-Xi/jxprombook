# 1. PHP-FPM php-fpm-exporter

[bakins/php-fpm-exporter](https://github.com/bakins/php-fpm-exporter)

### **PHP-FPM max-children reached**

PHP-FPM reached max children - `{{ $labels.instance }}`

```
  - alert: Php-fpmMax-childrenReached
    expr: sum(phpfpm_max_children_reached_total) by (instance) > 0
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: PHP-FPM max-children reached (instance {{ $labels.instance }})
      description: "PHP-FPM reached max children - {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```