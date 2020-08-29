# solr-feasibilty-study-configoverlay
Feasibility study of how much of solrconfig.xml can be represented as configoverlay.json

There is often a discussion that solrconfig.xml should be very small with most of the configuration living in configoverlay.json and controlled by Config API: https://lucene.apache.org/solr/guide/8_6/config-api.html#config-api

This feasibility study will take Solr 8.6.1 default configuration and will review what is actually possible by trying to removing one section of solrconfig.xml at a time and putting it into configoverlay.json by using Config API itself.

## Initial setup:
1) cp -r .../solr-8.6.1/server/solr/_default/conf to configsets/original
2) cp .../solr-8.6.1/server/solr/solr.xml to solrhome/solr.xml
3) .../solr-8.6.1/bin/solr start -s solrhome
4) .../solr-8.6.1/bin/solr create_core -c original -d configsets/original
5) curl http://localhost:8983/solr/original/config > api-output/original.json

## Step 01: Reduce managed-schema to minimize files
Since I am focusing on solrconfig.xml and schema API, I don't need a really functional managed-schema and all the related resources. I just need for Config API to be the same. 

1) cp -r configsets/original configsets/01-minischema
2) xml ed -L -d "//comment()" configsets/01-minischema/managed-schema
3) Edit to Leave just id field, uniqueKey and fieldType it needs
4) Delete all the resource files as they are no longer referenced
5) .../solr-8.6.1/bin/solr create_core -c 01-minischema -d configsets/01-minischema/
6) This fails as \_version\_ field is also needed
7) Then it fails because it needs the field types required by the schemaless mode configuration in solrconfig.xml. Which also requires stopwords and synonym files as well.
8) curl http://localhost:8983/solr/01-minischema/config > api-output/01-minischema.json
9) diff api-output/01-minischema.json api-output/original.json

No difference, this means we can proceed.

## Step 02: Strip all conmments from solrconfig.xml
### Setup
There are so many comments in solrconfig.xml, it is nearly impossible to figure out what is happening there. 
1) cp -r configsets/01-minischema/ configsets/02-nocomments
2) xml ed -L -d "//comment()" configsets/02-nocomments/solrconfig.xml
3) create core; get config
4) diff api-output/02-nocomments.json api-output/original.json

It is still identical, now we can actually compare file and API output side-by-side.

### Observations:
#### luceneMatchVersion
`<luceneMatchVersion>8.6.1</luceneMatchVersion>` => ` "luceneMatchVersion":"org.apache.lucene.util.Version:8.6.1"`

1. The config API has injected a class name here. Why?
2. It does not seem that it is possible to change that value, which is reasonable given its very low level impact.
3. This element will need to stay in solrconfig.xml
