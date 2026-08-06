```bash
# From literal key-value pairs
kubectl create configmap mysql-config \
  --from-literal=MYSQL_HOST=mysql.dev.local \
  --from-literal=MYSQL_PORT=3306 \
  --from-literal=MYSQL_DATABASE=mydb

# From a single file
kubectl create configmap mysql-config --from-file=config.txt
# Key --> config.txt
# Value --> file content

# From multiple files
kubectl create configmap mysql-config \
  --from-file=db.conf \
  --from-file=app.conf

# From a directory
kubectl create configmap mysql-config --from-file=./config-dir/

# From specific key=filename mapping
kubectl create configmap mysql-config --from-file=custom_key=db.conf
# Key name becomes custom_key instead of filename

# From env file (.env format)
kubectl create configmap mysql-config --from-env-file=.env

# Combine multiple sources
kubectl create configmap mysql-config \
  --from-literal=ENV=dev \
  --from-file=config.txt \
  --from-env-file=.env

# Dry-run (generate YAML without creating)
kubectl create configmap mysql-config \
  --from-literal=MYSQL_HOST=mysql.dev.local \
  --dry-run=client -o yaml

# Create + save YAML to file
kubectl create configmap mysql-config \
  --from-literal=MYSQL_HOST=mysql.dev.local \
  --dry-run=client -o yaml > configmap.yaml

# Create in specific namespace
kubectl create configmap mysql-config \
  --from-literal=MYSQL_HOST=mysql.dev.local \
  -n dev

# Update (since create won’t overwrite)
kubectl create configmap mysql-config \
  --from-literal=MYSQL_HOST=newhost \
  --dry-run=client -o yaml | kubectl apply -f -
```