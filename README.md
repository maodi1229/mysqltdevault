# mysqltdevault

## Description

MYSQL Community Edition plugin for Transparent Data Encryption (TDE). 

Master Key storage method： Communicates with HashiCorp Vault for backend storage.

You can use the compiled files directly, or you can use the source code to make changes and then compile your own plugin!

* Compiled plugins are placed in： `build/keyring_vault.so`

* If you want to compile your own plugin, you need to download the mysql source code first, and then place this current project under the plugin directory of your mysql project. (rename as `keyring_vault`)

mysql project plugin directory look like as below:
   
![image](https://github.com/maodi1229/mysqltdevault/assets/56705346/3468f31a-77ca-4c34-8aa5-77b008782dcf)

For information on how to compile the mysql plugin, see the official mysql documentation from https://dev.mysql.com/doc/extending-mysql/5.7/en/compiling-plugin-libraries.html

## how to use keyring_vault.so

### 1. HashiCorp Vault Setup

* Fetch the HashiCorp Vault binary.

Download the HashiCorp Vault binary appropriate for your platform from https://www.vaultproject.io/downloads.html.

* Create the HashiCorp Vault server configuration file.

Prepare a configuration file named `config.hcl` with the following content. For the tls_cert_file, tls_key_file, and path values, substitute path names appropriate for your system.

```js

listener "tcp" {
  address="127.0.0.1:8200"
  tls_cert_file="/home/username/certificates/vault.crt"
  tls_key_file="/home/username/certificates/vault.key"
}

storage "file" {
  path = "/home/username/vaultstorage/storage"
}

ui = true


# default 1.5 * 365 days, please adjust this value according to your actual situation
default_lease_ttl = "13140h"

# 3 * 365 days = 26280h, please adjust this value according to your actual situation
max_lease_ttl = "26280h"

```

For additional details, please see Hashicorp's official documentation from https://developer.hashicorp.com/vault/docs/configuration.

The following steps assumes that you've already got the `Root Token` through Vault Server initialization.

### 2. HashiCorp Vault Config

Login http://yourvaultserverip:8200/ui with root token obtained from the previous step.

**Note: You can do all of the following steps from the command line, the following demonstration is only via the UI**

#### 2.1 Create Secret Engine, you can replace `tdestore` with another name that makes sense.
![image](https://github.com/maodi1229/mysqltdevault/assets/56705346/9ed98592-5a47-4a42-b620-49855be52ff9)


#### 2.2 Create ACL Policy, `tdestore` needs to be consistent with the naming in the `Create Secret Engine` step
![image](https://github.com/maodi1229/mysqltdevault/assets/56705346/efa47a6e-aac0-4bdf-8f13-8c208a9c21f2)

```javascript
path "tdestore/*" {
   capabilities = ["list"]
}

path "tdestore/dc/*" {
   capabilities = ["create", "read", "delete", "update", "list"]
}
```

#### 2.3 Create token with ACL Policy to access Secret Engine 

Depending on your version of Vault, you may need to launch the `API Explorer` through the Web Cli
![Untitled](https://github.com/maodi1229/mysqltdevault/assets/56705346/176b9647-fafd-40bc-a5d9-8dbb77298311)

**Create token through `API Explorer`**

```diff
# This token needs to be configured in the mysql plugin, which we'll talk about later.
- Please note in particular that tokens have a lifecycle, depending on a few settings
- You need to renew the token at regular intervals to ensure that it is valid.
+ For more information on vault tokens, see https://developer.hashicorp.com/vault/docs/concepts/tokens
```

API Endpoint: 

```js
POST /auth/token/create

body:{
  "display_name": "mysqlt001",
  "no_parent": true,
  "period": "700h",
  "policies": [
    "tdestore"
  ],
  "renewable": true,
  "type": "service"
}
```
![image](https://github.com/maodi1229/mysqltdevault/assets/56705346/71d5aecb-2d1b-4939-82f3-bf9737e9075f)

The following `client_token` is the result we want

```js
{
  "request_id": "353c7e3b-cf25-3617-cca5-5c2064376f97",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": null,
  "wrap_info": null,
  "warnings": null,
  "auth": {
    "client_token": "hvs.CAESILHdRmHRXSwiiNTJBZWF1eBClR4jALAa3FgIi5HiwRKFGigKImh2cy44SFpmQVI3QkhXTGVMOXpTbUxoT2N1Nk0ubDBuOTMQyJ8D",
    "accessor": "Nh48tOTwDJ7bNaafO6zhYbMn.l0n93",
    "policies": [
      "default",
      "tdestore"
    ],
    "token_policies": [
      "default",
      "tdestore"
    ],
    "metadata": null,
    "lease_duration": 2520000,
    "renewable": true,
    "entity_id": "",
    "token_type": "service",
    "orphan": true,
    "mfa_requirement": null,
    "num_uses": 0
  }
}
```

### 3. MYSQL Config

#### 3.1 Find out the plugin directory 

```sh
SHOW VARIABLES LIKE 'plugin_dir';
```

Place the `keyring_vault.so` plugin into the plugin directory. 

#### 3.2 MySQL 8 Configuration parameters

create `keyring_vault.conf` file and place under /var/lib/mysql-keyring/ directory
```js
vault_url = http://yourvaultserverip:8200
secret_mount_point = tdestore/dc/master
token = hvs.CAESILHdRmHRXSwiiNTJBZWF1eBClR4jALAa3FgIi5HiwRKFGigKImh2cy44SFpmQVI3QkhXTGVMOXpTbUxoT2N1Nk0ubDBuOTMQyJ8D
vault_ca = /etc/vault_ca/vault.pem  #depending on your vault server enable ca or not
```

MYSQL configuration
```sh
early-plugin-load="keyring_vault=keyring_vault.so"
loose-keyring_vault_config="/var/lib/mysql-keyring/keyring_vault.conf"
binlog_encryption = ON
default_table_encryption = ON
innodb_redo_log_encrypt = ON
innodb_undo_log_encrypt = ON
```

#### 3.3 Make sure MySQL is allowed to send http request
```sh
setsebool -P mysql_connect_http 1
```

restart mysql server, that's all!










