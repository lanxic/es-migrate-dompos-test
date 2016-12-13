#### es-migrate-dompos-test #
========

index migration (migrate and rollback) for Elasticsearch index.


### Installation ###
---------------

Clone the project repository via bitbucket:
```
git@bitbucket.org:lanxic/alex-devops-test.git
```

### Basic Usage and Examples ###
------------------------
Running es-migrate-dompos with -h option will give you list of option that es-migrate-dompos supports.

```
Usage: ./es-migrate-dompos.sh [OPTIONS]

Where OPTIONS:
  -a HOST       Specify URL address of the Elasticsearch server
  -b            Rollback mode
  -c FILE       Specify path of config file
  -e ENVIRON    Environment name, default should be “production”
  -h            Printing the help
  -m NAME       Specify the name of migration file
  -o FILE       save log output to the FILE
  -r            Dry run mode
  -v            Print the version
```
this option I call from http://github.com/astasoft/shgrate it's to much lazy more thinking option

* Prepare the elasticsearch for the migration
you can use docker for fast deployment just put
```
docker pull elasticsearch
```

* Configure the docker for connect to public port so user can be access 9200 or 9300 port
```
sudo docker run --restart=always --name elasticsearch-test -p 9200:9200 -p 9300:9300 -d elasticsearch
```

* create the directories for es-migrate-dompos
```
mkdir migrations migrated rollback
```
this same with `http://github.com/astasoft/shgrate`

* Create the migration file
```
$ ./es-migrate-dompos.sh -m create_index_user
```
before you test please make sure in list index elasticsearch not all ready
you can be check with this `curl 'http://localhost:9200/_cat/indices?v'`

after you run command in above this tool will create file in directory migrations and rollback
put like this sample statement to create index "books".
```
$ cat migrations/2016_09_27_01_34_04_create_index_user.sg_migrate.esm
## es-migrate-dompos Migration Script
## Generated by: es-migrate-dompos v1.0
## File: 2016_09_27_01_34_04_create_index_user.sg_migrate.esm
## Date:
## Write your INDEX migration below this line

curl -XPUT '{{ES_SERVER}}/books/' -d '{
"settings": {
    "number_of_shards": 1
},
"mappings": {
    "book": {
        "properties": {
            "book_id": {
                "type": "string",
                "index": "not_analyzed"
            },
            "title": {
                "type": "string"
            }
        }
    }
}
}
'

$ cat rollback//2016_09_27_01_34_04_create_index_user.sg_migrate.esm
## es-migrate-dompos Rollback Script
## Generated by: es-migrate-dompos v1.0
## File: 2016_09_27_01_34_04_create_index_user.sg_migrate.esm
## Date:
## Write your INDEX rolllback migration below this line

curl -XDELETE '{{ES_SERVER}}/books/'
```

* Run the migration
Before running the migration it's good practice to see what es-migrate-dompos would
execute by running in dry run mode using -r option. Option -a tells es-migrate-dompos
the name of elasticsearch server to use.
```
$ ./es-migrate-dompos.sh -a http://localhost:9200 -r
ceksg http://localhost:9200
Migrating 2016_09_27_01_34_04_create_index_user.sg_migrate.esm...done.
>> Contents of file migrations/2016_09_27_01_34_04_create_index_user.sg_migrate.esm:
## es-migrate-dompos Migration Script
## Generated by: es-migrate-dompos v1.0
## File: 2016_09_27_01_34_04_create_index_user.sg_migrate.esm
## Date:
## Write your INDEX migration below this line

curl -XPUT 'http://localhost:9200/books/' -d '{
"settings": {
    "number_of_shards": 1
},
"mappings": {
    "book": {
        "properties": {
            "book_id": {
                "type": "string",
                "index": "not_analyzed"
            },
            "title": {
                "type": "string"
            }
        }
    }
}
}
'
```
if you think it ok and correct now you can execute to production
```
$ ./es-migrate-dompos.sh -a http://localhost:9200
ceksg http://localhost:9200
Migrating 2016_09_27_01_34_04_create_index_user.sg_migrate.esm...done.
```
you can be check with this `curl 'http://localhost:9200/_cat/indices?v'` look index books successfully Created
```
$ curl 'http://localhost:9200/_cat/indices?v'
health status index pri rep docs.count docs.deleted store.size pri.store.size
yellow open   books   1   1          0            0       130b           130b
```

* Rollback the changes
Doing the rollback is almost the same as doing the migration, you just need to
use -b option. Let's try to rollback the changes that we've done before but
let's do it in dry run mode first.
```
$ ./es-migrate-dompos.sh -a http://localhost:9200 -b -r
ceksg http://localhost:9200
Rollback 2016_09_27_01_34_04_create_index_user.sg_migrate.esm...done.
>> Contents of file migrated/production/2016_09_27_01_34_04_create_index_user.sg_migrate.esm:
## es-migrate-dompos Rollback Script
## Generated by: es-migrate-dompos v1.0
## File: 2016_09_27_01_34_04_create_index_user.sg_migrate.esm
## Date:
## Write your INDEX rolllback migration below this line

curl -XDELETE 'http://localhost:9200/books/'
```

Everything seems as expected do the real rollback.
```
$ ./es-migrate-dompos.sh -a http://localhost:9200 -b
ceksg http://localhost:9200
Rollback 2016_09_27_01_34_04_create_index_user.sg_migrate.esm...done.
```

### THX TO ###
es-migrate-dompos.sh written under reuse from shgrate(https://github.com/astasoft/shgrate)
because Iam to Lazy more thinking whats should be I use
