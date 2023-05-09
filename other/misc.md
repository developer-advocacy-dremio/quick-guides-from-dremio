## Misc Notes

- Using Nessie with an External Database Store:
```
docker run --rm --network host -p 19120:19120 -e NESSIE_VERSION_STORE_TYPE=JDBC \
  -e NESSIE_VERSION_STORE_PERSIST_JDBC_CATALOG=nessie \
  -e NESSIE_VERSION_STORE_PERSIST_JDBC_SCHEMA=nessie \
  -e QUARKUS_DATASOURCE_USERNAME=<user-name> \
  -e QUARKUS_DATASOURCE_PASSWORD=<password> \
  -e QUARKUS_DATASOURCE_JDBC_URL=jdbc:postgresql://localhost.example.com:5432/nessie \
  ghcr.io/projectnessie/nessie:0.58.0
```
[citation - Iceberg Slack](https://apache-iceberg.slack.com/archives/C025PH0G1D4/p1683612467672079)
