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

INSERT INTO simple_columnar SELECT generate_series(1,100000);
\d+ simple_columnar
```

The access method of the `simple_columnar` table must be `columnar`.
