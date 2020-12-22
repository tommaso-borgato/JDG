# Intro

EAP version must be at least 7.4.0.CD21: with 7.4.0.CD20 you still have to use jboss-datagrid-7.3.8-server since `remote-cache-container` doesn't have the necessary `properties` needed to configure security;
In case you want to use EAP version prior to 7.4.0.CD21, you have to disable JDG security (note there is no cli):
sed -i "s/_NODE_IDENTIFIER_/WFL2/g" $WLF_CLI_SCRIPT_TMP_2

# create RHDG user

This user is needed because the access to RHDG is password protected by default (`./redhat-datagrid-8.1.1-server/server/conf/infinispan.xml`: `<endpoints socket-binding="default" security-realm="default">`):

```
./redhat-datagrid-8.1.1-server/bin/cli.sh user create jbosseap -p jbosseap
```

# start RHDG

```
./redhat-datagrid-8.1.1-server/bin/server.sh -c infinispan.xml --cluster-stack=tcp --node-name=jdg1 --bind-address=$(hostname -I | cut -d' ' -f1)
```

# configure EAP

This is inspired by [InfinispanServerSetupTask.java](https://github.com/wildfly/wildfly/blob/master/testsuite/integration/clustering/src/test/java/org/jboss/as/test/clustering/cluster/web/remote/InfinispanServerSetupTask.java):

```
cat <<EOT > wf.cli
embed-server --std-out=echo --server-config=standalone-ha.xml
/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=infinispan-server:add(port=11222,host=192.168.1.15)
batch
/subsystem=infinispan/remote-cache-container=jdg_rc:add(default-remote-cluster=infinispan-server-cluster, module=org.wildfly.clustering.web.hotrod, properties={infinispan.client.hotrod.auth_username=jbosseap, infinispan.client.hotrod.auth_password=jbosseap}, protocol-version=3.0, statistics-enabled=true)
/subsystem=infinispan/remote-cache-container=jdg_rc/remote-cluster=infinispan-server-cluster:add(socket-bindings=[infinispan-server])
run-batch
/subsystem=infinispan/cache-container=web/invalidation-cache=jdg_ic:add()
/subsystem=infinispan/cache-container=web/invalidation-cache=jdg_ic/store=hotrod:add(remote-cache-container=jdg_rc,fetch-state=false,preload=false,passivation=false,purge=false,shared=true)
/subsystem=infinispan/cache-container=web:write-attribute(name=default-cache, value=jdg_ic)
EOT

./jboss-eap-7.4/bin/jboss-cli.sh --file=wf.cli
```

# start EAP

```
./jboss-eap-7.4/bin/standalone.sh --server-config=standalone-ha.xml -Djboss.default.jgroups.stack=tcp -Dprogram.name=wfl1 -Djboss.node.name=wfl1
```

# deploy to EAP

```
cp ../clusterbench-ee8.ear ./jboss-eap-7.4/standalone/deployments/
```

# TEST

```
curl http://localhost:8080/clusterbench/session
```