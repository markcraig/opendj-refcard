# ForgeRock DS/OpenDJ Refcard
_1.0 for server 5.5/4.1_

This card demonstrates common administrative and user tasks.

Command examples run in Bash, using short options where available.
Use `--help` for detailed usage.

Settings shown here are appropriate for evaluation on a single host system.
For production deployments, read the documentation at
<https://backstage.forgerock.com/docs/ds>.

## Getting the Software

### Download ForgeRock DS
https://backstage.forgerock.com/downloads

### Install Directory Server
```
cd /path/to ; unzip ~/Downloads/DS-5.5.zip
./opendj/setup directory-server -D "cn=Directory Manager" -w password -h ldap.example.com -p 1389 -Z 1636 --httpPort 8080 --httpsPort 8443 --adminConnectorPort 4444 -b dc=example,dc=com -l /path/to/Example.ldif --acceptLicense
```

### Install Directory Proxy
```
cd /path/to ; unzip ~/Downloads/DS-5.5.zip
# Proxy for OpenDJ/ForgeRock DS:
./opendj/setup proxy-server -D "cn=Directory Manager" -w password -h ldap.example.com -p 1389 -Z 1636 --adminConnectorPort 4444 --replicationServer rs.example.com:4444 --replicationBindDN "uid=admin,cn=Administrators,cn=admin data" --replicationBindPassword password --proxyUserBindDn cn=Proxy,ou=Apps,dc=example,dc=com --proxyUserBindPassword password --acceptLicense -X
# Proxy for generic LDAPv3:
./opendj/setup proxy-server -D "cn=Directory Manager" -w password -h ldap.example.com -p 1389 -Z 1636 --adminConnectorPort 4444 --staticPrimaryServer remote.example.com:1389 --baseDn dc=example,dc=com --proxyUserBindDn cn=Proxy,ou=Apps,dc=example,dc=com --proxyUserBindPassword password --acceptLicense
```

### Uninstall Server
```
# First unconfigure replication, if configured.
/path/to/opendj/bin/dsreplication unconfigure -h ldap.example.com -p 4444 -D "cn=Directory Manager" -w password --unconfigureAll -X -n
/path/to/opendj/bin/stop-ds ; rm -rf /path/to/opendj
```

## Managing Servers
```
export PATH=/path/to/opendj/bin:$PATH
```

| Task            | Command         |
|-----------------|-----------------|
| Start GUI       | `control-panel` |
| Start server    | `start-ds`      |
| Stop server     | `stop-ds`       |
| Restart server  | `stop-ds -R`    |

### Data

#### Enable Referential Integrity
```
dsconfig set-plugin-prop -h localhost -p 4444 -D "cn=Directory Manager" -w password --plugin-name "Referential Integrity" --set enabled:true -X -n
```

#### Create Backend
```
dsconfig create-backend -h localhost -p 4444 -D "cn=Directory Manager" -w password --backend-name userRoot -t je --set enabled:true --set base-dn:dc=example,dc=com --set db-cache-percent:50 -X -n
```

#### Generate LDIF
```
makeldif -o /path/to/generated.ldif /path/to/opendj/config/MakeLDIF/example.template
```

#### Import LDIF
```
import-ldif -h localhost -p 4444 -D "cn=Directory Manager" -w password -l /path/to/generated.ldif -n userRoot -t 0 -X
```

#### Backup All Data
```
backup -h localhost -p 4444 -D "cn=Directory Manager" -w password -a -d /path/to/opendj/bak -t 0 -X
```

#### Export LDIF
```
export-ldif -h localhost -p 4444 -D "cn=Directory Manager" -w password -l /path/to/export.ldif -n userRoot -t 0 -X
```

#### Restore All Backends
```
restore -h localhost -p 4444 -D "cn=Directory Manager" -w password -d /path/to/opendj/bak -t 0 -X
```

#### Delete Backend
```
dsconfig delete-backend -h localhost -p 4444 -D "cn=Directory Manager" -w password --backend-name userRoot -X -n
```

### Indexes

#### Create Standard Index
```
dsconfig create-backend-index -h localhost -p 4444 -D "cn=Directory Manager" -w password --backend-name userRoot --index-name description --set index-type:equality -X -n
```

#### Rebuild an Index
```
rebuild-index -h localhost -p 4444 -D "cn=Directory Manager" -w password -b dc=example,dc=com --index description -t 0 -X
```

#### Rebuild All Indexes
```
rebuild-index -h localhost -p 4444 -D "cn=Directory Manager" -w password -b dc=example,dc=com --rebuildAll -t 0 -X
```

#### Verify Indexes
```
stop-ds -Q ; verify-index -b dc=example,dc=com --countErrors
```

### Passwords

#### Read User's Password Policy
```
ldapsearch -h localhost -p 1389 -D "cn=Directory Manager" -w password -b dc=example,dc=com "(uid=bjensen)" pwdPolicySubentry
```

#### Assign Password Admin Privilege
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: uid=kvaughan,ou=People,dc=example,dc=com
changetype: modify
add: ds-privilege-name
ds-privilege-name: password-reset
```

#### Reset Password
```
ldappasswordmodify -h localhost -p 1389 -D uid=kvaughan,ou=people,dc=example,dc=com -w bribery -a dn:uid=scarter,ou=People,dc=example,dc=com -n changeit
```

#### Reset Forgotten Root DN Password
```
stop-ds
encode-password -s PBKDF2 -i
Please enter the password :
Please renter the password:
Encoded Password:  "{PBKDF2}10000:***"
# On cn=Directory Manager,cn=Root DNs,cn=config,
# set userPassword: {PBKDF2}10000:***
vi /path/to/opendj/config/config.ldif
start-ds
```

#### Reset Global Admin Password
```
ldappasswordmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password -a "dn:cn=admin,cn=Administrators,cn=admin data" -n changeit
```

### Accounts

#### Assign Account Admin Rights
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: ou=People,dc=example,dc=com
changetype: modify
add: aci
aci: (target="ldap:///ou=People,dc=example,dc=com")
 (targetattr ="*||+")(version 3.0;
  acl "Allow access to change all attributes";
  allow(all) userdn =
  "ldap:///uid=kvaughan,ou=People,dc=example,dc=com";)
```

#### Lock Account
```
manage-account set-account-is-disabled -h localhost -p 4444 -D uid=kvaughan,ou=people,dc=example,dc=com -w bribery -X -b uid=scarter,ou=People,dc=example,dc=com -O true
```

#### Unlock Account
```
manage-account set-account-is-disabled -h localhost -p 4444 -D uid=kvaughan,ou=people,dc=example,dc=com -w bribery -X -b uid=scarter,ou=People,dc=example,dc=com -O false
```

#### Enable Email Account Notifications
```
dsconfig set-global-configuration-prop -h localhost -p 4444 -D "cn=Directory Manager" -w password --set smtp-server:smtp.example.com -X -n
dsconfig set-account-status-notification-handler-prop -h localhost -p 4444 -D "cn=Directory Manager" -w password --handler-name "SMTP Handler" --set enabled:true --set email-address-attribute-type:mail -X -n
dsconfig set-password-policy-prop -h localhost -p 4444 -D "cn=Directory Manager" -w password --policy-name "Default Password Policy" --set account-status-notification-handler:"SMTP Handler" -X -n
```

### Replication

#### Basic Two-Server Setup
```
cd /path/to ; unzip ~/Downloads/DS-5.5.zip ; cp -R opendj replica
./opendj/setup directory-server -D "cn=Directory Manager" -w password -h ldap.example.com -p 1389 -Z 1636 --adminConnectorPort 4444 -b dc=example,dc=com -l /path/to/Example.ldif --acceptLicense
./replica/setup directory-server -D "cn=Directory Manager" -w password -h ldap.example.com -p 2389 -Z 3636 --adminConnectorPort 5444 -b dc=example,dc=com -a --acceptLicense
```

#### Configure Replication
```
dsreplication configure --adminUID admin --adminPassword password --baseDn dc=example,dc=com --host1 ldap.example.com --port1 4444 --bindDN1 "cn=Directory Manager" --bindPassword1 password --replicationPort1 8989 --host2 ldap.example.com --port2 5444 --bindDN2 "cn=Directory Manager" --bindPassword2 password --replicationPort2 9989 -X -n
```

#### Initialize Replication
```
dsreplication initialize-all -h ldap.example.com -p 4444 -I admin -w password -b dc=example,dc=com -X -n
```

#### Check Replication Status
```
dsreplication status -h ldap.example.com -p 4444 -I admin -w password -X -n
```

#### Restore Replica From Backup
```
dsreplication pre-external-initialization -h ldap.example.com -p 4444 -I admin -w password -b dc=example,dc=com -X -n
restore -h localhost -p 4444 -D "cn=Directory Manager" -w password -d /path/to/opendj/bak -t 0 -X
dsreplication post-external-initialization -h ldap.example.com -p 4444 -I admin -w password -b dc=example,dc=com -X -n
```

#### Read Changelog
```
ldapsearch -h localhost -p 1389 -D "cn=Directory Manager" -w password -b cn=changelog -J "1.3.6.1.4.1.26027.1.5.4:false" "(&)" changes changeLogCookie targetDN
```

#### Suspend/Resume Replication
```
dsreplication suspend -h ldap.example.com -p 4444 -I admin -w password -X -n
dsreplication resume -h ldap.example.com -p 4444 -I admin -w password -X -n
```

#### Unconfigure Replication
```
dsreplication unconfigure -h ldap.example.com -p 4444 -D "cn=Directory Manager" -w password --unconfigureAll -X -n
dsreplication unconfigure -h ldap.example.com -p 5444 -D "cn=Directory Manager" -w password --unconfigureAll -X -n
```

### Default Logs

| Log file                       | Description                 |
|--------------------------------|-----------------------------|
| `logs/errors`                  | Server errors and info      |
| `logs/http-access.audit.json`  | Common HTTP access          |
| `logs/ldap-access.audit.json`  | Common LDAP access          |
| `logs/replication`             | Replication repair          |
| `logs/server.out`              | Server events since startup |
| `logs/server.pid`              | Server process ID           |

### Monitoring

#### Read Monitor Info
```
# LDAP
ldapsearch -p 1389 -D "cn=Directory Manager" -w password -b cn=monitor "(&)" \* +
# REST
curl --user kvaughan:bribery http://localhost:8080/admin/monitor
```

#### Check Server Status
```
status -D "cn=Directory Manager" -w password -X
```

#### Enable Email Alerts
```
dsconfig set-global-configuration-prop -h localhost -p 4444 -D "cn=Directory Manager" -w password --set smtp-server:smtp.example.com -X -n
dsconfig create-alert-handler -h localhost -p 4444 -D "cn=Directory Manager" -w password --handler-name "SMTP Alert Handler" -t smtp --set enabled:true --set message-subject:"Directory Server Alert, Type: %%alert-type%%, ID: %%alert-id%%" --set message-body:"%%alert-message%%" --set recipient-address:admin@example.com --set sender-address:opendj@example.com -X -n
```

### Security

#### Import Trusted Cert
```
keytool -import -alias ca-cert -file ca.crt -trustcacerts -keystore /path/to/opendj/config/keystore -storepass:file /path/to/opendj/config/keystore.pin -storetype PKCS12 -noprompt
```

#### Change Key Pair/Cert Alias
```
keytool -genkeypair -alias new-cert -ext "san=dns:ldap.example.com" -dname "CN=ldap.example.com,O=Example Corp,C=FR" -keystore /path/to/opendj/config/keystore -storetype PKCS12 -storepass:file /path/to/opendj/config/keystore.pin -keypass:file /path/to/opendj/config/keystore.pin
dsconfig set-connection-handler-prop -h localhost -p 4444 -D "cn=Directory Manager" -w password --handler-name "LDAPS Connection Handler" --set ssl-cert-nickname:new-cert -X -n
dsconfig set-connection-handler-prop -h localhost -p 4444 -D "cn=Directory Manager" -w password --handler-name "LDAPS Connection Handler" --set enabled:false -X -n
dsconfig set-connection-handler-prop -h localhost -p 4444 -D "cn=Directory Manager" -w password --handler-name "LDAPS Connection Handler" --set enabled:true -X -n
```

### Troubleshooting

#### Cancel Task
```
# Display task IDs
manage-tasks -h localhost -p 4444 -D "cn=Directory Manager" -w password -s -X
# Cancel by ID
manage-tasks -h localhost -p 4444 -D "cn=Directory Manager" -w password -X -c <taskID>
```

#### Enter Lockdown Mode
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: ds-task-id=Enter Lockdown Mode,cn=Scheduled Tasks,cn=tasks
objectClass: top
objectClass: ds-task
ds-task-id: Enter Lockdown Mode
ds-task-class-name: org.opends.server.tasks.EnterLockdownModeTask
```

#### Leave Lockdown Mode
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: ds-task-id=Leave Lockdown Mode,cn=Scheduled Tasks,cn=tasks
objectClass: top
objectClass: ds-task
ds-task-id: Leave Lockdown Mode
ds-task-class-name: org.opends.server.tasks.LeaveLockdownModeTask
```

#### Enable Debug Logging
```
dsconfig set-log-publisher-prop -h localhost -p 4444 -D "cn=Directory Manager" -w password --publisher-name "File-Based Debug Logger" --set enabled:true -X -n
dsconfig create-debug-target -h localhost -p 4444 -D "cn=Directory Manager" -w password --publisher-name "File-Based Debug Logger" -t generic --target-name org.opends.server.api --set enabled:true -X -n
tail -f /path/to/opendj/logs/debug
```

#### Log TLS/SSL Debug Info
```
OPENDJ_JAVA_ARGS="-Djavax.net.debug=all" stop-ds -R
```


## Using Client Applications

```
export PATH=/path/to/opendj/bin:$PATH
```

### LDAP

#### Simple Search Filter
```
ldapsearch -h localhost -p 1389 -b dc=example,dc=com "(uid=bjensen)"
```

#### Complex Search Filter
```
ldapsearch -h localhost -p 1389 -b dc=example,dc=com "(&(sn=Jensen)(l=San Francisco))"
```

#### Search Attribute Lists
```
ldapsearch -h localhost -p 1389 -b dc=example,dc=com "(uid=bjensen)" mail
ldapsearch -h localhost -p 1389 -b dc=example,dc=com "(uid=bjensen)" +
ldapsearch -h localhost -p 1389 -b dc=example,dc=com "(uid=bjensen)" @person
ldapsearch -h localhost -p 1389 -b dc=example,dc=com "(uid=bjensen)" 1.1
```

#### Search Filter Operators

| Operator        | Desription                                   |
|-----------------|----------------------------------------------|
| `=`             | Equality, as in `(sn=Jensen)`                |
| `<=`            | Less than or equal to, alphanumeric          |
| `>=`            | Greater than or equal to, alphanumeric       |
| `=*`            | Presence, e.g. all entries with userPassword match `(userPassword=*)`. |
| `~=`            | Approximate, "sounds like"                   |
| `[:dn][:oid]:=` | Extensible match, e.g. `(ou:dn:=people)` matches all entries with `ou=people` in the DN or in the entry. E.g. `(lastLoginTime:1.3.6.1.4.1.26027.1.4.5:=-13w)` entries with last login in last 13 weeks. Add digits to OID: 1. (`<`), .2 (`<=`), .3 (`=`, default), .4 (`>=`), .5 (`>`), .6 (substring). |
| `!`             | NOT, as in `!(objectclass=person)`           |
| `&`             | AND, as in `(&(sn=Jensen)(l=San Francisco))` |
| `\|`            | OR, as in `(\|(sn=Jensen)(sn=Johnson))`       |

#### Search Filter Escape Characters
| Replace           | With    |
|-------------------|---------|
| `*`               | `\2a`   |
| `(`               | `\28`   |
| `)`               | `\29`   |
| `\`               | `\5c`   |
| `NUL (0x00)`      | `\00`   |

#### Add Entry
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: cn=Arsene Lupin,ou=Special Users,dc=example,dc=com
objectClass: person
objectClass: top
cn: Arsene Lupin
telephoneNumber: +33 1 23 45 67 89
sn: Lupin
```

#### Add Attribute Value
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: cn=Arsene Lupin,ou=Special Users,dc=example,dc=com
changetype: modify
add: description
description: Gentleman thief and master of disguise
```

#### Replace Attribute Value
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: cn=Arsene Lupin,ou=Special Users,dc=example,dc=com
changetype: modify
replace: description
description: Fictional character
```

#### Delete Attribute
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: cn=Arsene Lupin,ou=Special Users,dc=example,dc=com
changetype: modify
delete: description
```

#### Delete Single Attribute Value
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: uid=bjensen,ou=People,dc=example,dc=com
changetype: modify
delete: cn
cn: Babs Jensen
```

#### Move Entry
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: cn=Arsene Lupin,ou=Special Users,dc=example,dc=com
changetype: moddn
newrdn: cn=Arsene Lupin
deleteoldrdn: 0
newsuperior: ou=People,dc=example,dc=com
```

#### Rename Entry
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: uid=scarter,ou=people,dc=example,dc=com
changetype: modrdn
newrdn: uid=sjensen
deleteoldrdn: 1

dn: uid=sjensen,ou=people,dc=example,dc=com
changetype: modify
replace: cn
cn: Sam Jensen
-
replace: sn
sn: Jensen
-
replace: homeDirectory
homeDirectory: /home/sjensen
-
replace: mail
mail: sjensen@example.com
```

#### Delete Entry
```
ldapdelete -h localhost -p 1389 -D "cn=Directory Manager" -w password "cn=Arsene Lupin,ou=People,dc=example,dc=com"
```

#### Change Own Password
```
ldappasswordmodify -h localhost -p 1389 -a dn:uid=bjensen,ou=people,dc=example,dc=com -c hifalutin -n changeit
```

#### Read Schema
```
ldapsearch -h localhost -p 1389 -b cn=schema -s base "(&)" +
```

#### Read Server Capabilities
```
ldapsearch -h localhost -p 1389 -b "" -s base "(&)" +
```

#### Create Static Group
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: cn=My Static Group,ou=Groups,dc=example,dc=com
cn: My Static Group
objectClass: groupOfNames
objectClass: top
ou: Groups
member: uid=ahunter,ou=People,dc=example,dc=com
member: uid=bjensen,ou=People,dc=example,dc=com
member: uid=tmorris,ou=People,dc=example,dc=com
```

#### Create Dynamic Group
```
ldapmodify -h localhost -p 1389 -D "cn=Directory Manager" -w password
dn: cn=My Dynamic Group,ou=Groups,dc=example,dc=com
cn: My Dynamic Group
objectClass: top
objectClass: groupOfURLs
ou: Groups
memberURL: ldap:///ou=People,dc=example,dc=com??sub?l=San Francisco
```

#### Read Group Membership
```
ldapsearch -h localhost -p 1389 -b dc=example,dc=com "(uid=bjensen)" isMemberOf
```

### REST

#### Enable `/api`
```
dsconfig set-http-endpoint-prop -h localhost -p 4444 -D "cn=Directory Manager" -w password --endpoint-name /api --set authorization-mechanism:"HTTP Basic" --set config-directory:config/rest2ldap/endpoints/api --set enabled:true -X -n
```

#### Create
```
curl --request PUT --user kvaughan:bribery --header "Content-Type: application/json" --header "If-None-Match: *" --data '{"_id":"newuser", "_schema":"frapi:opendj:rest2ldap:user:1.0", "contactInformation": {"telephoneNumber":"+1 408 555 1212", "emailAddress":"newuser@example.com"}, "name": {"familyName":"New", "givenName":"User"}, "displayName":["New User"], "manager": {"_id":"kvaughan", "displayName":"Kirsten Vaughan"}}' http://localhost:8080/api/users/newuser
```

#### Read
```
curl --user kvaughan:bribery http://localhost:8080/api/users/newuser
```

#### Update
```
curl --request PUT --user kvaughan:bribery --header "Content-Type: application/json" --header "If-Match: *" --data '{"contactInformation": {"telephoneNumber":"+1 408 555 4798", "emailAddress":"scarter@example.com"}, "name": {"familyName":"Carter", "givenName":"Sam"}, "userName":"scarter@example.com", "displayName": ["Sam Carter", "Samantha Carter"], "groups": [{"_id":"Accounting Managers"}], "manager": {"_id":"trigden", "displayName":"Torrey Rigden"}, "uidNumber":1002, "gidNumber":1000, "homeDirectory":"/home/scarter"}' http://localhost:8080/api/users/scarter
```

#### Delete
```
curl --request DELETE --user kvaughan:bribery http://localhost:8080/api/users/newuser
```

#### Patch
```
curl --user kvaughan:bribery --request PATCH --header "Content-Type: application/json" --data '[{ "operation": "replace", "field": "/contactInformation/emailAddress", "value": "babs@example.com" }]' http://localhost:8080/api/users/bjensen
```

#### Action: Change Own Password
```
curl --request POST --user bjensen:hifalutin --header "Content-Type: application/json" --insecure --data '{ "oldPassword": "hifalutin", "newPassword": "changeme"}' https://localhost:8443/api/users/bjensen?_action=modifyPassword
```

#### Query
```
curl --user kvaughan:bribery http://localhost:8080/api/users?_queryFilter=userName+eq+'babs@example.com'
```

### LDAP Search Filters vs. REST Query Filters

| LDAP Search Filter                 | REST Query Filter                                          |
|------------------------------------|------------------------------------------------------------|
| `(&)`                              | `_queryFilter=true`                                        |
| `(uid=*)`                          | `_queryFilter=_id+pr`                                      |
| `(uid=bjensen)`                    | `_queryFilter=_id+eq+'bjensen'`                            |
| `(uid=*jensen*)`                   | `_queryFilter=_id+co+'jensen'`                             |
| `(uid=jensen*)`                    | `_queryFilter=_id+sw+'jensen'`                             |
| `(&(uid=*jensen*)(cn=babs*))`      | `_queryFilter=(_id+co+'jensen'+and+displayName+sw+'babs')` |
| `(\|(uid=*jensen*)(cn=sam*))`       | `_queryFilter=(_id+co+'jensen'+or+displayName+sw+'sam')`   |
| `(!(uid=*jensen*))`                | `_queryFilter=!(_id+co+'jensen')`                          |
| `(uid<=jensen)`                    | `_queryFilter=_id+le+'jensen'`                             |
| `(uid>=jensen)`                    | `_queryFilter=_id+ge+'jensen'`                             |

* * * * *

Copyright 2017 ForgeRock AS.

<https://github.com/markcraig/opendj-refcard>
