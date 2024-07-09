# Gitlab 

Install gitlab with external postgres and minio.

create database before install gitlab

create minio bucket before install gitlab

## Installation

```bash
helm install gitlab gitlab/gitlab -n gitlab -f <CONFIG_VALUES_FILE>
```