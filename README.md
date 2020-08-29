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
