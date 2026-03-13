```bash
# Create airflow user
docker exec -it airflow-webserver airflow users create \
  --username admin \
  --firstname Admin \
  --lastname User \
  --role Admin \
  --email admin@example.com \
  --password admin
```

```bash
docker compose pull
docker compose up -d
docker compose down
```
