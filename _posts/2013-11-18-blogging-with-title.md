# ad-crud-service

This guide was wriiten assuming that you are already familiar with Active Directory, how nodejs works, (if you havent you should go do it right now) and you need to create a microservice that create/read/update/delete enitities in your Active direcotry using the ldap protocol and ldapjs library.
For more advanced solution I will add support for Promises using Bluebird and Seneca for creating our service.

## Ldapjs Getting started
The first thing that we need to do is to create our ldap client. In our initial version we will use a simple json configuration. After that we will use the dot-env library to securely access our environment variables.


### Create ldapjs client

```javascript
const ldap = require('ldapjs');

/**
 * For more info and examples check http://ldapjs.org/client.html
 */
class LDAPClient{
    constructor(options){
        this.config = options;            
        this.ldapClient = this.initClient(options);
    }
    
    /**
     * Performs an add operation against the LDAP server.
     * Allows you to add an entry (which is just a plain JS object), and as always, controls are optional.
     */
    create(dn, entry, callback) {
        this.ldapClient.add(dn, entry, callback);
    };
    
    /**
     * Performs a search operation against the LDAP server.
     * Options can be a string representing a valid LDAP filter.
     */
    read(dn, options, callback) {
        this.ldapClient.search(dn, options, function (err, res) {
            var object = null;
            res.on('searchEntry', function (entry) {
                object = entry.object;
            });
            res.on('error', function (error) {
                callback(error);
            });
            res.on('end', function (result) {
                callback(result.status != 0 ? result : null, object);
            });
        });
    };

    /**
     * Performs an LDAP modify operation against the LDAP server. This API requires you to pass in a Change object, which is described below. Note that you can pass in a single Change or an array of Change objects.
     */
    update(dn, changes, callback) {
        this.modify(dn, changes, callback);
    };

    /**
     * Deletes an entry from the LDAP server.
     */
    delete(dn, callback) {
        this.client.del(dn, callback);
    };


    initClient(config){
        if (!this.ldapClient) {
            var options = {
                url: config.config.ldapServerUrl,
                bindDN: config.config.ldapUser,
                bindCredentials: config.config.ldapPassword,
                tlsOptions: {
                    secureOptions: config.config.secureOptions,
                    ciphers: config.config.sslCiphers
                },
                reconnect: true
            }
    
            return ldap.createClient(options);
        }
        return ldapClient;
    }
}

module.exports = LDAPClient;

```

### Configure application
```javascript
exports.config = {
    ldapServerUrl: 'ldap://host:port',
    ldapUser: 'testUser',
    ldapPassword: "pass123",
    adSearchAttributeName: 'sAMAccountName',
    usersContainerDn: 'DC=corp,DC=example',
};

```

[Checkout the source code](https://github.com/vgudzhev/ad-crud-service/tree/create-ldapjs-client "Source code")
