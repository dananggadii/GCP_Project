
# How to Migrate Database from VM to Cloud SQL for PostgreSQL on GCP

## 1. Prepare the Source Database for Migration

### Install and Configure `pglogical` Extension

```bash
sudo apt install postgresql-13-pglogical

sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/pg_hba_append.conf ."
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/postgresql_append.conf ."
sudo su - postgres -c "cat pg_hba_append.conf >> /etc/postgresql/13/main/pg_hba.conf"
sudo su - postgres -c "cat postgresql_append.conf >> /etc/postgresql/13/main/postgresql.conf"

sudo systemctl restart postgresql@13-main
```

### Enable Extension on Databases

```bash
sudo su - postgres
psql
```

```sql
\c postgres;
CREATE EXTENSION pglogical;

\c orders;
CREATE EXTENSION pglogical;

\c gmemegen_db;
CREATE EXTENSION pglogical;

\l
```

### Create Migration User

```sql
CREATE USER migration_admin PASSWORD 'DMS_1s_cool!';
ALTER DATABASE orders OWNER TO migration_admin;
ALTER ROLE migration_admin WITH REPLICATION;
```

### Grant Permissions on `pglogical` Schema and Tables

#### For `postgres` Database

```sql
\c postgres;
GRANT USAGE, ALL ON SCHEMA pglogical TO migration_admin;
GRANT SELECT ON ALL TABLES IN SCHEMA pglogical TO migration_admin;
```

#### For `orders` Database

```sql
\c orders;
GRANT USAGE, ALL ON SCHEMA pglogical TO migration_admin;
GRANT USAGE, ALL ON SCHEMA public TO migration_admin;

GRANT SELECT ON pglogical.tables TO migration_admin;
GRANT SELECT ON pglogical.depend TO migration_admin;
GRANT SELECT ON pglogical.local_node TO migration_admin;
GRANT SELECT ON pglogical.local_sync_status TO migration_admin;
GRANT SELECT ON pglogical.node TO migration_admin;
GRANT SELECT ON pglogical.node_interface TO migration_admin;
GRANT SELECT ON pglogical.queue TO migration_admin;
GRANT SELECT ON pglogical.replication_set TO migration_admin;
GRANT SELECT ON pglogical.replication_set_seq TO migration_admin;
GRANT SELECT ON pglogical.replication_set_table TO migration_admin;
GRANT SELECT ON pglogical.sequence_state TO migration_admin;
GRANT SELECT ON pglogical.subscription TO migration_admin;

GRANT SELECT ON public.distribution_centers TO migration_admin;
GRANT SELECT ON public.inventory_items TO migration_admin;
GRANT SELECT ON public.order_items TO migration_admin;
GRANT SELECT ON public.products TO migration_admin;
GRANT SELECT ON public.users TO migration_admin;
```

#### For `gmemegen_db` Database

```sql
\c gmemegen_db;
GRANT USAGE, ALL ON SCHEMA pglogical TO migration_admin;
GRANT USAGE, ALL ON SCHEMA public TO migration_admin;

GRANT SELECT ON ALL TABLES IN SCHEMA pglogical TO migration_admin;
GRANT SELECT ON public.meme TO migration_admin;
```

### Change Table Ownership in `orders`

```sql
\c orders;
ALTER TABLE public.distribution_centers OWNER TO migration_admin;
ALTER TABLE public.inventory_items OWNER TO migration_admin;
ALTER TABLE public.order_items OWNER TO migration_admin;
ALTER TABLE public.products OWNER TO migration_admin;
ALTER TABLE public.users OWNER TO migration_admin;
```

### Exit PostgreSQL Session

```sql
\q
exit
```

## 2. Create a Database Migration Service Connection Profile

1. Copy the internal IP address of the VM instance.

2. Go to Database Migration > Connection Profiles > Create Profile.

3. Set:
    - Profile role: Source
    - Database engine: PostgreSQL
    - Connection profile name: postgres-vm
    - Region: your region
    - Hostname or IP: (from step 1)
    - Port: 5432
    - Username: migration_admin
    - Password: DMS_1s_cool!

4. Click Create.

## 3. Create and Start a Continuous Migration Job

1. Go to Database Migration > Migration Jobs > Create Migration Job.

2. Set:
    - Migration job name: vm-to-cloudsql
    - Source database engine: PostgreSQL
    - Destination region: your region
    - Destination engine: Cloud SQL for PostgreSQL
    - Type: Continuous

3. Select:
    - Source profile: postgres-vm
    - Destination Instance ID: postgresql-cloudsql
    - Password: supersecret!
    - Edition: Enterprise
    - Version: PostgreSQL 13
    - Zone: your zone
    - Connectivity: Private IP

4. Choose:
    - Machine: 1 vCPU, 3.75 GB
    - Storage: SSD, 10 GB

5. Configure VPC Peering and click Continue.

### Allow Access to PostgreSQL VM

1. Go to VPC Network > VPC Network Peering.

2. Click servicenetworking-googleapis-com > Effective Routes View.

3. Copy IP range of peering-route-*.

4. Edit pg_hba.conf:

```bash
sudo nano /etc/postgresql/13/main/pg_hba.conf
# Replace 0.0.0.0/0 with the copied IP range
host all all <your IP> md5
```

5. Restart PostgreSQL:

```bash
sudo systemctl start postgresql@13-main
```

### Test and Start Migration

1. In Database Migration Service, click Test Job.

2. If successful, click Create & Start Job.

3. Monitor via Migration Jobs > vm-to-cloudsql.

## 4. Confirm Data in Cloud SQL

1. Go to SQL > Expand instance postgresql-cloudsql-master.

2. Click postgresql-cloudsql (replica) > Databases.

### Connect to Cloud SQL

```bash
gcloud sql connect postgresql-cloudsql --user=postgres --quiet
```

Password: supersecret!

```sql
\c orders;
select * from distribution_centers;
\q
```

### Update Source to Test Migration

```bash
export VM_NAME=postgresql-vm
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
export POSTGRESQL_IP=$(gcloud compute instances describe ${VM_NAME} \
  --zone=us-east4-b --format="value(networkInterfaces[0].accessConfigs[0].natIP)")
echo $POSTGRESQL_IP
psql -h $POSTGRESQL_IP -p 5432 -d orders -U migration_admin
```

Password: DMS_1s_cool!

```sql
\c orders;
insert into distribution_centers values(-80.1918,25.7617,'Miami FL',11);
\q
```

### Verify Data in Cloud SQL Again

```bash
gcloud sql connect postgresql-cloudsql --user=postgres --quiet
```

```sql
\c orders;
select * from distribution_centers;
\q
```

## 5. Promote Cloud SQL to Standalone

1. Go to Database Migration > Migration Jobs.

2. Click vm-to-cloudsql.

3. Click Promote.

4. Go to Databases > SQL to verify.

