# Ansible Playbook for Elasticsearch Mongo River
This is an [Ansible](http://www.ansibleworks.com/) role for configuring a river
from a mongodb collection to an elasticsearch index via the 
[elasticsearch-river-mongodb plugin](https://github.com/richardwilly98/elasticsearch-river-mongodb).
It is meant to be used in conjunction with this [Ansible Elasticsearch Playbook](https://github.com/Traackr/ansible-elasticsearch).

## Features
 * Configures the local mongo database to run as a replica set node.
 * Automatically regenerates Elasticsearch river index when its template or the river configuration changes.
 * If the river breaks, you can force the elasticsearch index to be regenerated
   by adding the parameter `--extra-vars "reindex=true"` when you run your playbook.

# Set-up Instructions:

### 0. This assumes you have a mongo instance deployed

### 1. Include this role and ansible-elasticsearch in your playbook

Checkout this project and ansible-elasticsearch as submodules under your ansible project's roles directory:

```
$  cd roles
$  git submodule add git://github.com/traackr/ansible-elasticsearch.git ./ansible-elasticsearch
$  git submodule add {{this repository}} ./ansible-elasticsearch-river-mongo
$  git submodule update --init
$  git commit ./submodule -m "Added submodules"
```

Then edit your main playbook file to contain these roles:

```yaml
---

- hosts: all_nodes
  user: ubuntu
  sudo: yes

  roles:
    # ansible elasticsearch must come first
    - ansible-elasticsearch
    - ansible-elasticsearch-river-mongo

  vars_files:
    - vars/my-vars.yml
```


### 2. Add the configuration variables required for ansible-elasticsearch and this role to group_vars/all.yml

These are the configuration variables we're currently using for ansible-elasticsearch.
You can find more information about them [here](https://github.com/Traackr/ansible-elasticsearch/).
**It is critical that you add elasticsearch-river-mongodb to the plugins.**

```
elasticsearch_version: 1.2.2
elasticsearch_heap_size: 2g
elasticsearch_max_open_files: 65535
elasticsearch_timezone: "America/New_York"
elasticsearch_node_max_local_storage_nodes: 1
elasticsearch_index_mapper_dynamic: "true"
elasticsearch_memory_bootstrap_mlockall: "true"
elasticsearch_max_locked_memory: unlimited
elasticsearch_plugins:
  - { name: 'elasticsearch/elasticsearch-mapper-attachments/2.2.1' }
  - { name: 'com.github.richardwilly98.elasticsearch/elasticsearch-river-mongodb/2.0.1' }
  - { name: 'facet-script', url: 'http://dl.bintray.com/content/imotov/elasticsearch-plugins/elasticsearch-facet-script-1.1.2.zip' }
elasticsearch_thread_pools:
  - "threadpool.bulk.type: fixed"
  - "threadpool.bulk.size: 50"
  - "threadpool.bulk.queue_size: 1000"
```

You will probably need to override these variables to point the river at your
mongodb collection and specify what you want your elasticsearch index to be called.
**Also, your mongo instance must be configured as a node in a replica set.**
Setting river_configure_mongo to yes will configure your local mongo instance
if you are running a compatible version.
(We are using a version from an ubuntu apt package).

```
river_configure_mongo: false
river_mongo_host: localhost
river_mongo_port: 27017
river_mongo_db_name: mongodb_name
river_collection: mongodb_collection
river_elasticsearch_index_name: item_index
river_elasticsearch_index_type: item
```

See the defaults folder for other river configuration variables available.

### 3. Now you can run your playbook

The playbook should print out the number of documents indexed
so far as debug information, and an error if no documents
were indexed.

Check the log file at `/var/log/elasticsearch/elasticsearch.log`
to see if anything went wrong with elasticsearch.

Restarting elasticsearch tends to break the river.
If this happens, use the reindex variable to create a new
river.

# License

Apache
