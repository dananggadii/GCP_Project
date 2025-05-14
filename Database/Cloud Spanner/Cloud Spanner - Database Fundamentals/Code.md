# Cloud Spanner - Database Fundamentals

## 3. Create a table in your database

```sql
CREATE TABLE Customer (
  CustomerId STRING(36) NOT NULL,
  Name STRING(MAX) NOT NULL,
  Location STRING(MAX) NOT NULL,
) PRIMARY KEY (CustomerId);
```

## 4. Insert and modify data

```sql
INSERT INTO
  Customer (CustomerId,
    Name,
    Location)
VALUES
  ('bdaaaa97-1b4b-4e58-b4ad-84030de92235',
    'Richard Nelson',
    'Ada Ohio'
    );
```

```sql
INSERT INTO
  Customer (CustomerId,
    Name,
    Location)
VALUES
  ('b2b4002d-7813-4551-b83b-366ef95f9273',
    'Shana Underwood',
    'Ely Iowa'
    );
```

```sql
SELECT * FROM Customer;
```

## 5. Use the Google Cloud CLI with Cloud Spanner

```bash
gcloud spanner instances create banking-instance-2 \
--config=regional-  \
--description="Banking Instance 2" \
--nodes=2
```

```bash
gcloud spanner instances list
```

```bash
gcloud spanner databases create banking-db-2 --instance=banking-instance-2
```

```bash
gcloud spanner instances update banking-instance-2 --nodes=1
```

```bash
gcloud spanner instances list
```

## 6. Use Automation Tools with Cloud Spanner

```bash 
terraform -version
```

```bash
nano spanner.tf
```

```bash
resource "google_spanner_instance" "banking-instance-3" {
  name         = "banking-instance-3"
  config       = "regional-"
  display_name = "Banking Instance 3"
  num_nodes    = 2
  labels = {
  }
}
```

```bash
terraform init
```

```bash
terraform plan
```

```bash
terraform apply
```

```bash
gcloud spanner instances list
```

## 7. Deleting instances

```bash
gcloud spanner instances delete banking-instance-2
```

```bash
gcloud spanner instances list
```

