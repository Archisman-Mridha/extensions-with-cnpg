Spinup the K3D cluster :
```sh
k3d cluster create --config ./k3d/config.yaml
```

Install CloudNative PG (CNPG) operator :
```sh
helm repo add cnpg https://cloudnative-pg.github.io/charts

helm repo update

helm upgrade --install cnpg \
  --version 0.26.0 \
  --namespace cnpg-system \
  --create-namespace \
  cnpg/cloudnative-pg
```

Build the Postgres container image :
```sh
docker build --platform linux/amd64 -t docker.io/archismanmridha/custom-postgres:17.0 ./postgres
docker push docker.io/archismanmridha/custom-postgres:17.0
```

Deploy the Postgres database cluster :
```sh
kubectl apply -f ./manifests/test.cluster.yaml
```

Ensure that the Citus extension works, by executing the following PostgreSQL queries inside the
`test-1` pod :
```sql
CREATE EXTENSION IF NOT EXISTS citus;

CREATE TABLE simple_columnar(i INT8) USING columnar;
INSERT INTO simple_columnar SELECT generate_series(1,100000);
\d+ simple_columnar
```

The access method of the `simple_columnar` table must be `columnar`.

## Backups

Install CertManager :
```sh
helm install \
  cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --version v1.19.1 \
  --namespace cert-manager --create-namespace \
  --set crds.enabled=true
```

Install the `Barman Cloud Plugin`, in the same namespace where the `CNPG operator` got installed :
```sh
kubectl apply \
  -f https://github.com/cloudnative-pg/plugin-barman-cloud/releases/download/v0.7.0/manifest.yaml \
  --namespace cnpg-system
```

Let's apply the manifests :
```sh
k apply \
  -f ./manifests/objectstore.yaml \
  -f ./manifests/s3-credentials.secret.yaml \
  -f ./manifests/scheduled-backup.yaml
```

## Recovery

Let's delete the existing CNPG cluster :
```sh
k delete \
  -f ./manifests/cluster.yaml \
  -f ./manifests/scheduled-backup.yaml
```

And try to restore the just taken immediate backup into a new cluster from  :
> Currently, it's a limitation of CNPG, that we can't do in-place cluster recovery.
> By that, I mean, we need to create a separate cluster, with a separate name (`recovered-test`),
> otherwise, we'll get the following error :
> **ERROR: WAL archive check failed for server recoveredCluster: Expected empty archive**
```
k apply -f ./manifests/recovered.cluster.yaml
```

Once recovered, you can SSH into the recovered database pod, spin up Postgres CLI, and execute the
following, verifying the recovery :
```sh
\d+ simple_columnar
```

## REFERENCEs

- [Backup types](https://docs.pgbarman.org/release/3.16.1/user_guide/concepts.html#backup-types)

- [WAL archiving and WAL streaming](https://docs.pgbarman.org/release/3.16.1/user_guide/concepts.html#wal-archiving-and-wal-streaming)

- [Recovery](https://docs.pgbarman.org/release/3.16.1/user_guide/concepts.html#recovery)

- [[Bug]: Scheduled backup does not work with plugin type](https://github.com/cloudnative-pg/cloudnative-pg/issues/8445)

- [[Feature]: In-place recovery from backup](https://github.com/cloudnative-pg/cloudnative-pg/issues/5203)
