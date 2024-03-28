# Recommended Labels

The metadata is organized around the concept of an _application_.

Shared labels and annotations share a common prefix: `app.kubernetes.io`. Labels without a prefix are private to users. The shared prefix ensures that shared labels do not interfere with custom user labels.



| Key | Description | Example | Type |
|:---:|:-----------:|:-------:|:----:|
| `app.kubernetes.io/name` | The name of the application | `mysql` | string |
| `app.kubernetes.io/instance` | A unique name identifying the instance of an application | `mysql-abcxzy` | string |
| `app.kubernetes.io/version` | The current version of the application (e.g., a SemVer 1.0, revision hash, etc.) | `5.7.21` | string |
| `app.kubernetes.io/component` | The component within the architecture | `database` | string |
| `app.kubernetes.io/part-of` | The name of a higher level application this one is part of | `wordpress` | string |
| `app.kubernetes.io/managed-by` | The tool being used to manage the operation of an application | `helm` | string |
