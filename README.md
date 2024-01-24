# mysqltdevault
MYSQL Community Edition plugin - enable Transparent Data Encryption (TDE). Communicates with HashiCorp Vault for back end storage.

You can use the compiled files directly, or you can use the source code to make changes and then compile your own plugin!

1. Compiled plugins are placed in： `build/keyring_vault.so`
2. If you want to compile your own plugin, you need to download the mysql source code first, and then place this current project under the plugin directory of your mysql project. (rename as `keyring_vault`)

   mysql project plugin directory look like as below:
   
![image](https://github.com/maodi1229/mysqltdevault/assets/56705346/3468f31a-77ca-4c34-8aa5-77b008782dcf)

# how to use keyring_vault.so


