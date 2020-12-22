# configure RHDG

Create the caches needed by the application we are going to deploy on EAP:

```
cat <<EOT > jdg.cli
embed-server --server-config=clustered.xml
/subsystem=transactions:write-attribute(name=node-identifier,value=jdg1)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8.ear.a.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8.ear.b.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8.ear.c.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8.ear.d.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8.ear.clusterbench-ee8-web.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8.ear.clusterbench-ee8-web-passivating.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8.ear.clusterbench-ee8-web-granular.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8-1.ear.clusterbench-ee8-web.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8-1.ear.clusterbench-ee8-web-passivating.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8-1.ear.clusterbench-ee8-web-granular.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8-2.ear.clusterbench-ee8-web.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8-2.ear.clusterbench-ee8-web-passivating.war:add(configuration=default)
/subsystem=datagrid-infinispan/cache-container=clustered/distributed-cache=clusterbench-ee8-2.ear.clusterbench-ee8-web-granular.war:add(configuration=default)
EOT

./jboss-datagrid-7.3.8-server/bin/cli.sh --file=jdg.cli
```

# start RHDG

```
./jboss-datagrid-7.3.8-server/bin/standalone.sh --server-config=clustered.xml -Djboss.default.jgroups.stack=tcp -Dprogram.name=jdg1 -Djboss.node.name=jdg1
```

# configure EAP:

This is inspired by [InfinispanServerSetupTask.java](https://github.com/wildfly/wildfly/blob/master/testsuite/integration/clustering/src/test/java/org/jboss/as/test/clustering/cluster/web/remote/InfinispanServerSetupTask.java):

```
cat <<EOT > wf.cli
embed-server --std-out=echo --server-config=standalone-ha.xml
/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=infinispan-server:add(port=11222,host=127.0.0.1)
batch
/subsystem=infinispan/remote-cache-container=jdg_rc:add(default-remote-cluster=infinispan-server-cluster, module=org.wildfly.clustering.web.hotrod, protocol-version=2.9, statistics-enabled=true)
/subsystem=infinispan/remote-cache-container=jdg_rc/remote-cluster=infinispan-server-cluster:add(socket-bindings=[infinispan-server])
run-batch
/subsystem=infinispan/cache-container=web/invalidation-cache=jdg_ic:add()
/subsystem=infinispan/cache-container=web/invalidation-cache=jdg_ic/store=hotrod:add(remote-cache-container=jdg_rc,fetch-state=false,preload=false,passivation=false,purge=false,shared=true)
/subsystem=infinispan/cache-container=web:write-attribute(name=default-cache, value=jdg_ic)
EOT

./jboss-eap-7.3/bin/jboss-cli.sh --file=wf.cli
```

# start EAP

```
./jboss-eap-7.3/bin/standalone.sh --server-config=standalone-ha.xml -Djboss.default.jgroups.stack=tcp -Dprogram.name=wfl1 -Djboss.node.name=wfl1 -Djboss.socket.binding.port-offset=100
```

# deploy to EAP

```
cp ../clusterbench-ee8.ear ./jboss-eap-7.3/standalone/deployments/
```

# TEST

```
curl http://localhost:8080/clusterbench/session
```