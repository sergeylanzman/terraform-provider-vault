---
layout: "vault"
page_title: "Vault: vault_database_secrets_mount resource"
sidebar_current: "docs-vault-resource-database-secrets-mount"
description: |-
  Configures any number of database secrets engines under a single mount resource
---

# vault\_database\_secrets\_mount

Configure any number of database secrets engines under a single dedicated mount resource.

~> **Important** All data provided in the resource configuration will be
written in cleartext to state and plan files generated by Terraform, and
will appear in the console output when Terraform runs. Protect these
artifacts accordingly. See
[the main provider documentation](../index.html)
for more details.

## Caveats:
This resource will be replaced for any of the following conditions:

- A database engine block is removed
- The `name` for any configured database engine is changed

## Example Usage

```hcl
resource "vault_database_secrets_mount" "db" {
  path = "db"

  mssql {
    name           = "db1"
    username       = "sa"
    password       = "super_secret_1"
    connection_url = "sqlserver://{{username}}:{{password}}@127.0.0.1:1433"
    allowed_roles = [
      "dev1",
    ]
  }

  postgresql {
    name              = "db2"
    username          = "postgres"
    password          = "super_secret_2"
    connection_url    = "postgresql://{{username}}:{{password}}@127.0.0.1:5432/postgres"
    verify_connection = true
    allowed_roles = [
      "dev2",
    ]
  }
}

resource "vault_database_secret_backend_role" "dev1" {
  name    = "dev1"
  backend = vault_database_secrets_mount.db.path
  db_name = vault_database_secrets_mount.db.mssql[0].name
  creation_statements = [
    "CREATE LOGIN [{{name}}] WITH PASSWORD = '{{password}}';",
    "CREATE USER [{{name}}] FOR LOGIN [{{name}}];",
    "GRANT SELECT ON SCHEMA::dbo TO [{{name}}];",
  ]
}

resource "vault_database_secret_backend_role" "dev2" {
  name    = "dev2"
  backend = vault_database_secrets_mount.db.path
  db_name = vault_database_secrets_mount.db.postgresql[0].name
  creation_statements = [
    "CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';",
    "GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";",
  ]
}
```

## Argument Reference

The following arguments are supported for the Vault `mount`:

* `path` - (Required) Where the secret backend will be mounted

* `description` - (Optional) Human-friendly description of the mount

* `default_lease_ttl_seconds` - (Optional) Default lease duration for tokens and secrets in seconds

* `max_lease_ttl_seconds` - (Optional) Maximum possible lease duration for tokens and secrets in seconds

* `audit_non_hmac_response_keys` - (Optional) Specifies the list of keys that will not be HMAC'd by audit devices in the response data object.

* `audit_non_hmac_request_keys` - (Optional) Specifies the list of keys that will not be HMAC'd by audit devices in the request data object.

* `local` - (Optional) Boolean flag that can be explicitly set to true to enforce local mount in HA environment

* `options` - (Optional) Specifies mount type specific options that are passed to the backend

* `seal_wrap` - (Optional) Boolean flag that can be explicitly set to true to enable seal wrapping for the mount, causing values stored by the mount to be wrapped by the seal's encryption capability

* `external_entropy_access` - (Optional) Boolean flag that can be explicitly set to true to enable the secrets engine to access Vault's external entropy source

The following arguments are common to all database engines:

* `plugin_name` - (Optional) Specifies the name of the plugin to use.

* `verify_connection` - (Optional) Whether the connection should be verified on
  initial configuration or not.

* `allowed_roles` - (Optional) A list of roles that are allowed to use this
  connection.

* `root_rotation_statements` - (Optional) A list of database statements to be executed to rotate the root user's credentials.

* `data` - (Optional) A map of sensitive data to pass to the endpoint. Useful for templated connection strings.

Supported list of database secrets engines that can be configured:

* `cassandra` - (Optional) A nested block containing configuration options for Cassandra connections.  
  *See [Configuration Options](#cassandra-configuration-options) for more info*

* `couchbase` - (Optional) A nested block containing configuration options for Couchbase connections.  
  *See [Configuration Options](#couchbase-configuration-options) for more info*
 
* `elasticsearch` - (Optional) A nested block containing configuration options for Elasticsearch connections.  
  *See [Configuration Options](#elasticsearch-configuration-options) for more info*

* `hana` - (Optional) A nested block containing configuration options for SAP HanaDB connections.  
  *See [Configuration Options](#sap-hanadb-sap-configuration-options) for more info*
 
* `mongodb` - (Optional) A nested block containing configuration options for MongoDB connections.  
  *See [Configuration Options](#mongodb-configuration-options) for more info*

* `mongodbatlas` - (Optional) A nested block containing configuration options for MongoDB Atlas connections.  
  *See [Configuration Options](#mongodb-atlas-configuration-options) for more info*

* `mssql` - (Optional) A nested block containing configuration options for MSSQL connections.  
  *See [Configuration Options](#mssql-configuration-options) for more info*

* `mysql` - (Optional) A nested block containing configuration options for MySQL connections.  
  *See [Configuration Options](#mysql-configuration-options) for more info*

* `mysql_rds` - (Optional) A nested block containing configuration options for RDS MySQL connections.  
  *See [Configuration Options](#mysql-configuration-options) for more info*

* `mysql_aurora` - (Optional) A nested block containing configuration options for Aurora MySQL connections.  
  *See [Configuration Options](#mysql-configuration-options) for more info*

* `mysql_legacy` - (Optional) A nested block containing configuration options for legacy MySQL connections.  
  *See [Configuration Options](#mysql-configuration-options) for more info*

* `oracle` - (Optional) A nested block containing configuration options for Oracle connections.  
  *See [Configuration Options](#oracle-configuration-options) for more info*

* `postgresql` - (Optional) A nested block containing configuration options for PostgreSQL connections.  
  *See [Configuration Options](#postgresql-configuration-options) for more info*

* `redshift` - (Optional) A nested block containing configuration options for AWS Redshift connections.  
  *See [Configuration Options](#aws-redshift-configuration-options) for more info*

* `snowflake` - (Optional) A nested block containing configuration options for Snowflake connections.  
  *See [Configuration Options](#snowflake-configuration-options) for more info*

* `influxdb` - (Optional) A nested block containing configuration options for InfluxDB connections.  
  *See [Configuration Options](#influxdb-configuration-options) for more info*
 
### Cassandra Configuration Options

* `hosts` - (Required) The hosts to connect to.

* `username` - (Required) The username to authenticate with.

* `password` - (Required) The password to authenticate with.

* `port` - (Optional) The default port to connect to if no port is specified as
  part of the host.

* `tls` - (Optional) Whether to use TLS when connecting to Cassandra.

* `insecure_tls` - (Optional) Whether to skip verification of the server
  certificate when using TLS.

* `pem_bundle` - (Optional) Concatenated PEM blocks configuring the certificate
  chain.

* `pem_json` - (Optional) A JSON structure configuring the certificate chain.

* `protocol_version` - (Optional) The CQL protocol version to use.

* `connect_timeout` - (Optional) The number of seconds to use as a connection
  timeout.

### Couchbase Configuration Options

* `hosts` - (Required) A set of Couchbase URIs to connect to. Must use `couchbases://` scheme if `tls` is `true`.

* `username` - (Required) Specifies the username for Vault to use.

* `password` - (Required) Specifies the password corresponding to the given username.

* `tls` - (Optional) Whether to use TLS when connecting to Couchbase.

* `insecure_tls` - (Optional) Whether to skip verification of the server
  certificate when using TLS.

* `base64_pem` - (Optional) Required if `tls` is `true`. Specifies the certificate authority of the Couchbase server, as a PEM certificate that has been base64 encoded.

* `bucket_name` - (Optional) Required for Couchbase versions prior to 6.5.0. This is only used to verify vault's connection to the server.

* `username_template` - (Optional) Template describing how dynamic usernames are generated.
 
### Elasticsearch Configuration Options

* `url` - (Required) The URL for Elasticsearch's API. https requires certificate
  by trusted CA if used.

* `username` - (Required) The username to be used in the connection.

* `password` - (Required) The password to be used in the connection.

### InfluxDB Configuration Options

* `host` - (Required) The host to connect to.

* `username` - (Required) The username to authenticate with.

* `password` - (Required) The password to authenticate with.

* `port` - (Optional) The default port to connect to if no port is specified as
  part of the host.

* `tls` - (Optional) Whether to use TLS when connecting to Cassandra.

* `insecure_tls` - (Optional) Whether to skip verification of the server
  certificate when using TLS.

* `pem_bundle` - (Optional) Concatenated PEM blocks configuring the certificate
  chain.

* `pem_json` - (Optional) A JSON structure configuring the certificate chain.

* `username_template` - (Optional) Template describing how dynamic usernames are generated.

* `connect_timeout` - (Optional) The number of seconds to use as a connection
  timeout.

### MongoDB Configuration Options

* `connection_url` - (Required) A URL containing connection information. See
  the [Vault
  docs](https://www.vaultproject.io/api-docs/secret/databases/mongodb.html#sample-payload)
  for an example.

* `username` - (Optional) The root credential username used in the connection URL.

* `password` - (Optional) The root credential password used in the connection URL.

* `username_template` - (Optional) For Vault v1.7+. The template to use for username generation.
See the [Vault
  docs](https://www.vaultproject.io/docs/concepts/username-templating)

### MongoDB Atlas Configuration Options

* `public_key` - (Required) The Public Programmatic API Key used to authenticate with the MongoDB Atlas API.

* `private_key` - (Required) The Private Programmatic API Key used to connect with MongoDB Atlas API.

* `project_id` - (Required) The Project ID the Database User should be created within.

### SAP HanaDB Configuration Options

* `connection_url` - (Required) A URL containing connection information. See
  the [Vault
  docs](https://www.vaultproject.io/api-docs/secret/databases/hanadb.html#sample-payload)
  for an example.

* `max_open_connections` - (Optional) The maximum number of open connections to
  use.

* `max_idle_connections` - (Optional) The maximum number of idle connections to
  maintain.

* `max_connection_lifetime` - (Optional) The maximum number of seconds to keep
  a connection alive for.

* `username` - (Optional) The root credential username used in the connection URL.

* `password` - (Optional) The root credential password used in the connection URL.

* `disable_escaping` - (Optional) Disable special character escaping in username and password.

### MSSQL Configuration Options

* `connection_url` - (Required) A URL containing connection information. See
  the [Vault
  docs](https://www.vaultproject.io/api-docs/secret/databases/mssql.html#sample-payload)
  for an example.

* `max_open_connections` - (Optional) The maximum number of open connections to
  use.

* `max_idle_connections` - (Optional) The maximum number of idle connections to
  maintain.

* `max_connection_lifetime` - (Optional) The maximum number of seconds to keep
  a connection alive for.

* `username_template` - (Optional) For Vault v1.7+. The template to use for username generation.
See the [Vault
  docs](https://www.vaultproject.io/docs/concepts/username-templating)

* `username` - (Optional) The root credential username used in the connection URL.

* `password` - (Optional) The root credential password used in the connection URL.

* `disable_escaping` - (Optional) Disable special character escaping in username and password.

* `contained_db` - (Optional bool: false) For Vault v1.9+. Set to true when the target is a
  Contained Database, e.g. AzureSQL.
  See the [Vault
  docs](https://www.vaultproject.io/api/secret/databases/mssql#contained_db)

### MySQL Configuration Options

* `connection_url` - (Required) A URL containing connection information. See
  the [Vault
  docs](https://www.vaultproject.io/api-docs/secret/databases/mysql-maria.html#sample-payload)
  for an example.

* `max_open_connections` - (Optional) The maximum number of open connections to
  use.

* `max_idle_connections` - (Optional) The maximum number of idle connections to
  maintain.

* `max_connection_lifetime` - (Optional) The maximum number of seconds to keep
  a connection alive for.

* `username` - (Optional) The root credential username used in the connection URL.

* `password` - (Optional) The root credential password used in the connection URL.

* `tls_certificate_key` - (Optional) x509 certificate for connecting to the database. This must be a PEM encoded version of the private key and the certificate combined.

* `tls_ca` - (Optional) x509 CA file for validating the certificate presented by the MySQL server. Must be PEM encoded.

* `username_template` - (Optional) For Vault v1.7+. The template to use for username generation.
See the [Vault
  docs](https://www.vaultproject.io/docs/concepts/username-templating)

### Oracle Configuration Options

* `connection_url` - (Required) A URL containing connection information. See
  the [Vault
  docs](https://www.vaultproject.io/api-docs/secret/databases/oracle.html#sample-payload)
  for an example.

* `max_open_connections` - (Optional) The maximum number of open connections to
  use.

* `max_idle_connections` - (Optional) The maximum number of idle connections to
  maintain.

* `username` - (Optional) The root credential username used in the connection URL.

* `password` - (Optional) The root credential password used in the connection URL.

* `max_connection_lifetime` - (Optional) The maximum number of seconds to keep
  a connection alive for.

* `username_template` - (Optional) For Vault v1.7+. The template to use for username generation.
  See the [Vault
  docs](https://www.vaultproject.io/docs/concepts/username-templating)

### PostgreSQL Configuration Options

* `connection_url` - (Required) A URL containing connection information. See
  the [Vault
  docs](https://www.vaultproject.io/api-docs/secret/databases/postgresql.html#sample-payload)
  for an example.

* `max_open_connections` - (Optional) The maximum number of open connections to
  use.

* `max_idle_connections` - (Optional) The maximum number of idle connections to
  maintain.

* `max_connection_lifetime` - (Optional) The maximum number of seconds to keep
  a connection alive for.

* `username` - (Optional) The root credential username used in the connection URL.

* `password` - (Optional) The root credential password used in the connection URL.

* `disable_escaping` - (Optional) Disable special character escaping in username and password.

* `username_template` - (Optional) For Vault v1.7+. The template to use for username generation.
See the [Vault
  docs](https://www.vaultproject.io/docs/concepts/username-templating)

### AWS Redshift Configuration Options

* `connection_url` - (Required) Specifies the Redshift DSN. See
  the [Vault
  docs](https://www.vaultproject.io/api-docs/secret/databases/redshift#sample-payload)
  for an example.

* `max_open_connections` - (Optional) The maximum number of open connections to
  the database.

* `max_idle_connections` - (Optional) The maximum number of idle connections to
  the database.

* `max_connection_lifetime` - (Optional) The maximum amount of time a connection may be reused.

* `username` - (Optional) The root credential username used in the connection URL.

* `password` - (Optional) The root credential password used in the connection URL.

* `disable_escaping` - (Optional) Disable special character escaping in username and password.

* `username_template` - (Optional) - [Template](https://www.vaultproject.io/docs/concepts/username-templating) describing how dynamic usernames are generated.


### Snowflake Configuration Options

* `connection_url` - (Required) A URL containing connection information. See
  the [Vault
  docs](https://www.vaultproject.io/api-docs/secret/databases/snowflake#sample-payload)
  for an example.

* `max_open_connections` - (Optional) The maximum number of open connections to
  use.

* `max_idle_connections` - (Optional) The maximum number of idle connections to
  maintain.

* `max_connection_lifetime` - (Optional) The maximum number of seconds to keep
  a connection alive for.

* `username` - (Optional) The username to be used in the connection (the account admin level).

* `password` - (Optional) The password to be used in the connection.

* `username_template` - (Optional) - [Template](https://www.vaultproject.io/docs/concepts/username-templating) describing how dynamic usernames are generated.

## Attributes Reference

* `engine_count` - The total number of database secrets engines configured.

## Import

Database secret backend connections can be imported using the `path` e.g.

```
$ terraform import vault_database_secrets_mount.db db
```
