# Helm chart for Microsoft SQL Server

Microsoft SQL Server is still used by many organizations. And migration to Cloud native can be a blocked by having this technology.
In order to help organization to lift-shift to Cloud Native, they can run also MS SQL in kubernetes via this Helm chart

> **DISCLAIMER**
> BY DEPLOYING THIS HELM CHART, YOU ARE ACCEPTING THE [END-USER Licensing Agreement of Microsoft SQL Server](https://github.com/microsoft/mssql-docker/blob/master/LICENSE)

## Install

Verify the helm chart via cosign

```sh
export COSIGN_PUBLIC_KEY=$(curl https://gitlab.com/xrow.keys | grep rsa | ssh-keygen -f /dev/stdin -e -m pem)
cosign verify --key env://COSIGN_PUBLIC_KEY registry.gitlab.com/xrow-public/helm-mssql/charts/mssql:1.1.3
```

values.yaml
```yaml
---
global:
  storageClass: "openebs-kernel-nfs" # Change this

auth:
  database: test
  rootPassword: 'yourStrong(#)Password'
  username: test
  password: 'yourStrong(#)Password'
```

```sh
helm upgrade --install mssql oci://registry.gitlab.com/xrow-public/helm-mssql/charts/mssql --version 1.1.3 --create-namespace -n mssql -f values.yaml
```

# values.yaml

Check default values of this chart [here](https://gitlab.com/xrow-public/helm-mssql/-/blob/main/chart/values.yaml) .

# Features

* Applying Helm Chart standards
* Ability to specify own registry
* Persisting data
* Auto Bootstrapping Database
* Auto Bootstraping Database Owner User with given password
* Ability to execute initial DB scripts (SQL)
* Performance Monitoring - Integrated with Prometheus Operator

## Data initalisation via init folder

```Docker
FROM registry.gitlab.com/xrow-public/helm-mssql:1.1.3

COPY *.sql /docker-entrypoint-initdb.d
```

## Test the container

```bash
podman build -t mssql .
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong#Passw0rd>' -p 1433:1433 -v sqlvolume:/var/opt/mssql -it localhost/mssql:1.1.3
```

## Credits

Credits to [ElmCompany](https://github.com/ElmCompany/helm-charts/tree/master/charts/mssql) for inital work.
