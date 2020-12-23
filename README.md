# JBoss EAP + RHDG Red Hat Data Grid

In this repository you can find some examples about running "JBoss EAP" connected to "Red Hat Data Grid - RHDG" for storing session data;

* [EAP-7.3](EAP-7.3): EAP-7.3 + jboss-datagrid-7.3.8-server
* [EAP-7.4](EAP-7.4): EAP-7.4 + redhat-datagrid-8.1.1-server
* [EAP-7.4-Hotrod](EAP-7.4-Hotrod): EAP-7.4 + redhat-datagrid-8.1.1-server using Hotrod
* [EAP-7.4-Encrypted](EAP-7.4-Encrypted): EAP-7.4 + redhat-datagrid-8.1.1-server using Hotrod and encrypted connection
* [WF](WF): WF + redhat-datagrid-8.1.1-server

All the examples assume you have downloaded and unzipped the necessary EAP and RHDG distributions using something like the following:

RHDG:

```
wget http://<RHDG_REPO>/redhat-datagrid-8.1.1.CR1-server.zip
rm -rdf /tmp/tests-clustering/redhat-datagrid-8.1.1-server/* /tmp/tests-clustering/redhat-datagrid-8.1.1-server/.[!.]*
mkdir -p /tmp/tests-clustering/redhat-datagrid-8.1.1-server
unzip -q redhat-datagrid-8.1.1.CR1-server.zip -d "/tmp/tests-clustering/"
```

EAP/WF:

```
wget http://<EAP_REPO>/jboss-eap-7.4.0.CD21-CR1.zip
rm -rdf /tmp/tests-clustering/jboss-eap-7.4/* /tmp/tests-clustering/jboss-eap-7.4/.[!.]*
mkdir -p /tmp/tests-clustering/jboss-eap-7.4
unzip -q jboss-eap-7.4.0.CD21-CR1.zip -d "/tmp/tests-clustering/"
```
