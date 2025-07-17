# Setup for the Arkeo Indexer

The **Arkeo Indexer** is a powerful two-part system that makes it easy to work with on-chain data from the Arkeo blockchain.
It consists of two key applications:

### The Indexer

A refactored and modernized version of Unchained, the Indexer connects directly to the Arkeo blockchain and parses on-chain data, storing it in a local PostgreSQL SQL database for fast, reliable queries. It keeps your database synced with the chain in near real-time, handling all the heavy lifting of extracting and organizing blockchain activity.

### The API
The API is a lightweight web service that sits on top of your SQL database. It exposes all the indexed blockchain data via easy-to-use REST endpoints, making it simple for dApps, explorers, dashboards, and other services to access up-to-date Arkeo data without worrying about parsing the chain themselves.

#### With the Indexer and API together, you can:
- Run your own private copy of all relevant Arkeo data.
- Serve fast, flexible, and custom queries to your users or apps.
- Stay in sync with the chain, with full control over data, uptime, and privacy.

## Indexer Setup

### Service Overview

The service used to launch the indexer contains any needed environmental variables and points at the config file.

```
[Unit]
Description=Arkeo Indexer Service
After=network-online.target

[Service]
User=user
WorkingDirectory=/home/user
ExecStart=/home/user/go/bin/indexer --config /home/user/arkeo_providers/indexer_config_1.yaml
Environment="LOG_LEVEL=info"
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

### Config File

This contains the core values needed for the running of the indexer.
```
arkeo_api: "http://localhost:1317"
tendermint_api: "http://localhost:26657"
tendermint_ws: "http://localhost:26657"
chain_id: "arkeo-main-v1"
bech32_pref_acc_addr: "arkeo"
bech32_pref_acc_pub: "arkeopub"
db:
  host: "localhost"                         # Use your database host or IP here
  port: 5432                                # Standard PostgreSQL port
  user: "arkeo_user"                        # Your database user
  pass: "REPLACE_WITH_SECRET_PASSWORD"      # <--- Set this securely!
  name: "arkeo"                             # Database name
  pool_max_conns: 20
  pool_min_conns: 1
  ssl_mode: "disable"                       # Set to "require" for production if possible
  connection_timeout: 30
```

### SQL Setup

You want to install Postgres SQL for the indexer.

With that installed, there is a setup for formatting the tables and adjusting access.

Make sure you did this command on your root arkeo directory from the command line:
```
make tools
```
This creates the TERN tool that is used to create the sql tables.

Do these commands on the server hosting the Postgres server. 

> Note that if you DROP DATABASE arkeo below, you will lose all your existing arkeo index data- but you probably know that.

```
psql -h 127.0.0.1 -U postgres -c "DROP DATABASE arkeo;"
psql -h 127.0.0.1 -U postgres -c "CREATE DATABASE arkeo;"
```

Update the tern config in the arkeo directory.

```
/Users/user/Projects/arkeo/directory/tern/tern.conf
```

```
[database]
host = "localhost"                          # Use your database host or IP here
port = 5432                                 # Standard PostgreSQL port
database = "arkeo"                          # Database name
user = "postgres"                           # Database user
password = "REPLACE_WITH_SECURE_PASSWORD"   # <--- Never commit real passwords!
version_table = "public.schema_version"
sslmode = "disable"                         # Use 'require' in production if possible

[data]
# Any fields in the data section are available in migration templates
# prefix = "foo"                            # Example custom field (uncomment as needed)
```

Use TERN to update the sql data structure. 

> Note, that you don't want to leave your sensitive Postgres admin data in this directory- so remember to delete this info once you setup your database.

Tern is a bit confusing to install as Arkeo uses this version: https://github.com/jackc/tern

```
go install github.com/jackc/tern/v2@latest
```

```
tern migrate -c /Users/user/Projects/arkeo/directory/tern/tern.conf -m /Users/user/Projects/arkeo/directory/tern
```

A long script of sql will run by as it updates your Postgres tables.

Now that there is a structure, you need to lower permissions for a user you want to use for the Indexer.

```
psql -h 100.28.199.0 -U postgres -d arkeo

CREATE USER arkeo_user WITH PASSWORD 'REPLACE_WITH_A_STRONG_PASSWORD';
```

No let's apply the user to the tables that TERN made.

```
psql -h 100.28.199.0 -U postgres -d arkeo

DO $$ DECLARE
r RECORD;
BEGIN
FOR r IN (SELECT tablename FROM pg_tables WHERE schemaname = 'public') LOOP
EXECUTE 'ALTER TABLE public.' || quote_ident(r.tablename) || ' OWNER TO arkeo_user;';
END LOOP;
END $$;

DO $$ DECLARE
r RECORD;
BEGIN
FOR r IN (SELECT sequence_name FROM information_schema.sequences WHERE sequence_schema = 'public') LOOP
EXECUTE 'ALTER SEQUENCE public.' || quote_ident(r.sequence_name) || ' OWNER TO arkeo_user;';
END LOOP;
END $$;

DO $$
DECLARE
r RECORD;
BEGIN
FOR r IN (SELECT table_name FROM information_schema.views WHERE table_schema = 'public') LOOP
EXECUTE 'ALTER VIEW public.' || quote_ident(r.table_name) || ' OWNER TO arkeo_user;';
END LOOP;
END $$;

ALTER SCHEMA public OWNER TO arkeo_user;
ALTER DATABASE arkeo OWNER TO arkeo_user;

\dt+

\q
```

### Testing

Now run your Indexer.

```
cd ~/go/bin  

./indexer --config /Users/user/arkeo_providers/indexer_config_1.yaml
```
And as that works, you will see blocks flying by and data in the Postgres sql.  If it looks good, then launch the background service.

## API Setup

### Service Overview

The service used to launch the api that exposes the indexer sql data on port 7777.

```
[Unit]
Description=Arkeo API Service
After=network-online.target

[Service]
User=user
WorkingDirectory=/home/user
ExecStart=/home/user/go/bin/api --config /home/user/arkeo_providers/api_config_1.yaml
Environment="LOG_LEVEL=info"
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

### Config File

This contains the core values needed for the running of the api.

```
db:
  host: "localhost"                     # Database host (update as needed)
  port: 5432                            # PostgreSQL port
  user: "arkeo_user"                    # Database user
  pass: "REPLACE_WITH_SECURE_PASSWORD"  # <-- Use a strong password!
  name: "arkeo"                         # Database name
  pool_max_conns: 5
  pool_min_conns: 1
  ssl_mode: "disable"                   # Use "require" for production if possible
  connection_timeout: 10
```
```
cd ~/go/bin  

./api --config /Users/user/arkeo_providers/api_config_1.yaml
```

### API Calls

Here are some calls for the Indexer API:

#### Health
```
http://127.0.0.1:7777/health
```

#### Stats
```
http://127.0.0.1:7777/stats
http://127.0.0.1:7777/stats/arkeo-mainnet-fullnode
```
#### Provider Search
```
http://127.0.0.1:7777/provider/search?service=arkeo-mainnet-fullnode&sort=contract_count
```

#### Provider
```
http://127.0.0.1:7777/provider/arkeopub1addwnpepqfn52r6xng2wwfrgz2tm5yvscq42k3yu3ky9cg3kw5s6p0qg7tfx75uwq3z?service=arkeo-mainnet-fullnode
```

#### Subscriber
```
http://127.0.0.1:7777/subscriber/arkeopub1addwnpepqtr4y6gahwhl3jxfahdnuf00s6fqw8he8rtssf8rf5gx2fwsc8ww7u93hf3?service=arkeo-mainnet-fullnode
```

## Provider Feedback

As a provider, you can tell us what more is needed with the index and api.

Join our community on Discord:

- Share your thoughts and suggestions. 
- Help newcomers who might be facing challenges you've already conquered. 
- Request features that you think could make Arkeo even better.

Your input helps shape the future of decentralized data, and we deeply appreciate your involvement.

Join our Data-Providers channel on Discord:
[Arkeo Discord, Data-Providers](https://discord.com/channels/1050100146626642052/1359893459854688439)
