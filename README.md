# mysqltdevault

## Description

MYSQL Community Edition plugin - enable Transparent Data Encryption (TDE). 

Master Key storage method： Communicates with HashiCorp Vault for backend storage.

You can use the compiled files directly, or you can use the source code to make changes and then compile your own plugin!

* Compiled plugins are placed in： `build/keyring_vault.so`

* If you want to compile your own plugin, you need to download the mysql source code first, and then place this current project under the plugin directory of your mysql project. (rename as `keyring_vault`)

mysql project plugin directory look like as below:
   
![image](https://github.com/maodi1229/mysqltdevault/assets/56705346/3468f31a-77ca-4c34-8aa5-77b008782dcf)

For information on how to compile the mysql plugin, see the official mysql documentation from https://dev.mysql.com/doc/extending-mysql/5.7/en/compiling-plugin-libraries.html

## how to use keyring_vault.so

### HashiCorp Vault Setup

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

```

For additional details, please see Hashicorp's official documentation from https://developer.hashicorp.com/vault/docs/configuration.

The following steps assumes that you've already got the `Root Token` through Vault Server initialization.

### HashiCorp Vault Config

Login http://yourvaultserverip:8200/ui with root token obtained from the previous step.

**Note: You can do all of the following steps from the command line, the following demonstration is only via the UI**

#### Create Secret Engine, you can replace `testing` with another name that makes sense.
![image](https://github.com/maodi1229/mysqltdevault/assets/56705346/876130d9-f169-44c6-9917-55b255d0e3f7)

#### Create ACL Policy, `testing` needs to be consistent with the naming in the `Create Secret Engine` step
![image](https://github.com/maodi1229/mysqltdevault/assets/56705346/00ba01c4-befd-46ed-9f49-2df8365fdc6f)
```javascript
path "testing/*" {
   capabilities = ["list"]
}

path "testing/dc1/*" {
   capabilities = ["create", "read", "delete", "update", "list"]
}
```

#### Create token with ACL Policy to access Secret Engine 

Depending on your version of Vault, you may need to launch the `API Explorer` through the Web Cli
![Untitled](https://github.com/maodi1229/mysqltdevault/assets/56705346/176b9647-fafd-40bc-a5d9-8dbb77298311)

**Create token through `API Explorer`**

This token needs to be configured in the mysql plugin, which we'll talk about later.



