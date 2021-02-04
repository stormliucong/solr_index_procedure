#### security configuration for solr. based on solr-8.8.0

1. Add a security.json in $SOLR_HOME/server/solr

```json
{
"authentication":{ 
   "blockUnknown": true, 
   "class":"solr.BasicAuthPlugin",
   "credentials":{"solr":"IV0EHq1OnNrj6gvRCwvFwTrZ1+z1oBbnQdiVC3otuq0= Ndd7LKvVBAaZIF0QAVi1ekCfAJXr1GGfLtRUXhgrF8c="}, 
   "realm":"My Solr users", 
   "forwardCredentials": false 
},
"authorization":{
   "class":"solr.RuleBasedAuthorizationPlugin",
   "permissions":[{"name":"security-edit",
      "role":"admin"}], 
   "user-role":{"solr":"admin"} 
}}
```

2. Create an user cl3720 with password yourpassword

```bash
curl --user solr:SolrRocks http://localhost:8983/solr/admin/authentication -H 'Content-type:application/json' -d '{"set-user": {"cl3720":"yourpassword"}}'
```

3. Open the updated security.json and change cl3720 to the admin. Remember to delete the user solr!
```json
{
    "authentication":
    {
        "blockUnknown": true,
        "class": "solr.BasicAuthPlugin",
        "credentials":
        {
            "cl3720": "keep this generated HASH unchanged"
        },
        "realm": "My Solr users",
        "forwardCredentials": false,
        "":
        {
            "v": 0
        }
    },
    "authorization":
    {
        "class": "solr.RuleBasedAuthorizationPlugin",
        "permissions": [
        {
            "name": "security-edit",
            "role": "admin"
        }],
        "user-role":
        {
            "cl3720": "admin"
        }
    }
}
```

4. Change $SOLR_HOME/bin/solr.in.sh to avoid 401 error by adding the following two lines.

```bash
SOLR_AUTH_TYPE="basic"
SOLR_AUTHENTICATION_OPTS="-Dbasicauth=cl3720:yourpassword"
```

5. Start solr with security enabled.

```bash
$SOLR_HOME/bin/solr start
```

6. Create the collection
```bash
$SOLR_HOME/./bin/solr create -c notes -s 2 -rf 2
```

7. Add custermized field. Since the original json is nested. using '.' to flat it out. 

```bash
# the original json time format is in ns long timestamp. solr only support pdate. I decided to go with plong for the timestamp. In the later client query, we can change date to timestamp for query.
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"TIME_STR_KEY.long", "type":"plong", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema 
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"PRIMARY_TIME.long", "type":"plong", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword@cumc http://localhost:8983/solr/notes/schema 
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"UPDATE_TIME.long", "type":"plong", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"MRN.string", "type":"string", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema 
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"EMPI.string", "type":"string", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema 
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"PATIENT_NAME.string", "type":"text_general", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema 
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"ORGANIZATION.int", "type":"pint", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema 
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"ALTERNATE_ID.string", "type":"string", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"EVENT_CODE.string", "type":"string", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"EVENT_STATUS.string", "type":"string", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"FACILITY_CODE.long", "type":"plong", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"TITLE.string", "type":"text_general", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema
# text_en has a nice support for english query in general.
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"TEXT.string", "type":"text_en", "multiValued":false, "indexed": true, "stored":true}}' -u cl3720:yourpassword http://localhost:8983/solr/notes/schema
```

8. Index 

```bash
$SOLR_HOME./bin/post -c notes $SOLR_HOME/example/notes/*.json -u cl3720:yourpassword
```

9. Enable SSL. SSL is not enabled for now since we want to limit the visits by localhost by now. In order to view Admin UI at http://localhost:8983/solr/ I have to establish a SSL tunnel between solr server and my local browser.

```bash
ssh -L 8983:local:8983 usernameforservernotsolrusername@hostname -N
```




