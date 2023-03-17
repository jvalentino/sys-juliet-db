# System Juliet Data Warehouse

This project represents Postgres Database as run on Kubernetes via Helm, as a part of the overall https://github.com/jvalentino/sys-juliet project. For system details, please see that location.

Prerequisites

- Git
- Helm
- Minikube
- psql
- Pgadmin

All of these you can get in one command using this installation automation (if you are on a Mac): https://github.com/jvalentino/setup-automation

## Stack

Postgres

> PostgreSQL, also known as Postgres, is a free and open-source relational database management system emphasizing extensibility and SQL compliance. It was originally named POSTGRES, referring to its origins as a successor to the Ingres database developed at the University of California, Berkeley

https://en.wikipedia.org/wiki/PostgreSQL

## Deployment

Prerequisites

- None

To re-install it, forward ports, and then verify it worked, use:

```bash
./deploy.sh
```

...which automatically runs the verification script to forward ports.

### deploy.sh

```bash
#!/bin/sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm delete pg-primary --wait || true
helm install pg-primary \
	--wait \
	--set auth.postgresPassword=postgres \
	--set auth.username=postgres \
	--set auth.database=examplesys \
	bitnami/postgresql
sh -x ./verify.sh
```

### verify.sh

```bash
#!/bin/sh
mkdir build || true
kubectl port-forward --namespace default svc/pg-primary-postgresql 5432:5432 > build/pg-primary-postgresql.log 2>&1 &
psql -d postgresql://postgres:postgres@localhost:5432/examplesys -c "select now()"

while [ $? -ne 0 ]; do
    kubectl port-forward --namespace default svc/pg-primary-postgresql 5432:5432 > build/pg-primary-postgresql.log 2>&1 &
    psql -d postgresql://postgres:postgres@localhost:5432/examplesys -c "select now()"
    sleep 5
done
```

## Runtime

### Kubernetes Dashboard

It will show on the Dashboard as a "Stateful Set" because this specific service involve storing data on the file system.

![01](https://github.com/jvalentino/sys-golf/raw/main/wiki/06.png)

### pgadmin

You can then verify it is working by using pgadmin:

[![01](https://github.com/jvalentino/sys-golf/raw/main/wiki/05.png)](https://github.com/jvalentino/sys-golf/blob/main/wiki/05.png)