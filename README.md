Following steps allow you to set up Grafana Monitoring for your oVirt 4.3 environment and use Grafana Dashboards from latest oVirt 4.4 on a previous release.

### Allowing Grafana to connect to oVirt DWH database (Data Warehouse)
Login to the oVirt engine 4.3 and create a user *"grafana"* with password *"grafana"* that will get a read-only access to the ovirt_engine_history database and will be able to use `public` schema
```
# su - postgres -c 'scl enable rh-postgresql10 bash'
# psql -U postgres -c "CREATE ROLE grafana WITH LOGIN ENCRYPTED PASSWORD 'grafana';" -d ovirt_engine_history
# psql -U postgres -c "GRANT CONNECT ON DATABASE ovirt_engine_history TO grafana;"
# psql -U postgres -c "GRANT USAGE ON SCHEMA public TO grafana;" ovirt_engine_history
```

Generate the rest of the permissions that will be granted to the newly created user and save them to a file:
```
# psql -U postgres -c "SELECT 'GRANT SELECT ON ' || relname || ' TO grafana;' FROM pg_class JOIN pg_namespace ON pg_namespace.oid = pg_class.relnamespace WHERE nspname = 'public' AND relkind IN ('r', 'v');" --pset=tuples_only=on  ovirt_engine_history > grant.sql
```
Use the file you created in the previous step to grant permissions to the newly created user:
```
# psql -U postgres -f grant.sql ovirt_engine_history
```
Remove the file you used to grant permissions to the newly created user:
```
# rm grant.sql
```
Exit the postgres user shell by pressing Ctrl+d

Add the following lines for the newly created user to /var/opt/rh/rh-postgresql10/lib/pgsql/data/pg_hba.conf preceding the line beginning local all all
```
host    ovirt_engine_history grafana 0.0.0.0/0               md5
host    ovirt_engine_history grafana :0/0                    md5
```

Reload postgres service
```
# systemctl reload rh-postgresql10-postgresql
```
### Installing Grafana
You can install Grafana directly on the oVirt Engine machine (this how it's done in oVirt 4.4) or on a separate machine. Following steps shows how you can install Grafana on a separate Oracle Linux 7 server. 
```
# yum-config-manager --enable ol7_optional_latest ol7_olcne11
# yum -y install grafana
# systemctl enable --now grafana-server
```
### Adding oVirt DWH database as Data Source in Grafana
Login to Grafana (default port 3000) and navigate to `Configuration` -> `DataSources` and click on green `Add Data Source` button. Select PostgreSQL source and use the following settings:
![Grafana Data Source](/images/ovirt-dwh-datasource.png)

### Importing Dashboards from

### Sources
https://www.ovirt.org/documentation/data_warehouse_guide/#Allowing_Read_Only_Access_to_the_History_Database

