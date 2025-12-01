# camel-hackathon-infra - PostgreSQL Chapter

The operator is installed already, so in your OCP project run.

```
oc create -f postgres.yaml
```

We connect to the database pod to create a table and add data to be extracted later.

```
oc rsh $(oc get pods -l postgres-operator.crunchydata.com/role=master -o name)
```

```
psql -U postgres test \
-c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO postgresadmin;"
```

Now let's change the password.

```
psql -U postgres -c "ALTER USER postgresadmin PASSWORD '<admin_pwd>';"
```

Now you can create your table and data, for example:

```
psql -U postgres test \
-c "CREATE TABLE test (data TEXT PRIMARY KEY);
INSERT INTO test(data) VALUES ('hello'), ('world');"
```

To check you can do

```
psql -U postgres test \
-c "SELECT * from test;"
```

and you should see

```
 data
-------
 hello
 world
(2 rows)
```
