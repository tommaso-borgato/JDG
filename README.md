# JBoss EAP + RHDG Red Hat Data Grid

In this repository you can find some examples about running "JBoss EAP" connected to "Red Hat Data Grid - RHDG" for storing session data;

* EAP-7.3 + jboss-datagrid-7.3.8-server
* EAP-7.4 + redhat-datagrid-8.1.1-server
* EAP-7.4-Encrypted: EAP-7.4 + redhat-datagrid-8.1.1-server
* WF + redhat-datagrid-8.1.1-server

All the examples assume you have downloaded and unzipped the necessary EAP and RHDG distributions using something like the following:

RHDG:

```
wget http://<RHDG_REPO>/redhat-datagrid-8.1.1.CR1-server.zip
rm -rdf /tmp/tests-clustering/jdg-1/* /tmp/tests-clustering/jdg-1/.[!.]*
mkdir -p /tmp/tests-clustering/jdg-1
unzip -q redhat-datagrid-8.1.1.CR1-server.zip -d "/tmp/tests-clustering/jdg-1/"
mv /tmp/tests-clustering/jdg-1/redhat-datagrid-8.1.1-server/* /tmp/tests-clustering/jdg-1/
mv /tmp/tests-clustering/jdg-1/redhat-datagrid-8.1.1-server/.[!.]* /tmp/tests-clustering/jdg-1/
rm -d /tmp/tests-clustering/jdg-1/redhat-datagrid-8.1.1-server
```

EAP/WF:

```
wget http://<EAP_REPO>/jboss-eap-7.4.0.CD21-CR1.zip
rm -rdf /tmp/tests-clustering/jboss-eap-1/* /tmp/tests-clustering/jboss-eap-1/.[!.]*
mkdir -p /tmp/tests-clustering/jboss-eap-1
unzip -q jboss-eap-7.4.0.CD21-CR1.zip -d "/tmp/tests-clustering/jboss-eap-1/"
mv /tmp/tests-clustering/jboss-eap-1/jboss-eap-7.4/* /tmp/tests-clustering/jboss-eap-1/
mv /tmp/tests-clustering/jboss-eap-1/jboss-eap-7.4/.[!.]* /tmp/tests-clustering/jboss-eap-1/
rm -d /tmp/tests-clustering/jboss-eap-1/jboss-eap-7.4
```
