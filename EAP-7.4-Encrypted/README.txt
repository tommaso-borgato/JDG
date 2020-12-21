# Intro

Connect EAP and RHDG using an encrypted connection;
The key-pair used for encryption is in the jdg.keystore.jks key-store;

EAP version must be at least 7.4.0.CD21: with 7.4.0.CD20 you still have to use jboss-datagrid-7.3.8-server since `remote-cache-container` doesn't have the necessary `properties` needed to configure security;
In case you want to use EAP version prior to 7.4.0.CD21, you have to disable JDG security (note there is no cli)

# Create the keystore and truststore:
Create keystore and truststore:
```
keytool -genkeypair -alias jdg-client -keyalg RSA -keysize 2048 -validity 365 -keystore /tmp/prova/jdg.keystore.jks -dname "CN=jdg.client" -keypass 123PIPPOBAUDO -storepass 123PIPPOBAUDO
keytool -exportcert -keystore /tmp/prova/jdg.keystore.jks -alias jdg-client -keypass 123PIPPOBAUDO -storepass 123PIPPOBAUDO -file /tmp/prova/jdg.cer
keytool -importcert -keystore /tmp/prova/jdg.truststore.jks -storepass 123PIPPOBAUDO -alias jdg-client -trustcacerts -file /tmp/prova/jdg.cer -noprompt
```
copy keystore and truststore in EAP and JDG folders:
```
cp /tmp/prova/jdg.keystore.jks ./redhat-datagrid-8.1.1-server/server/conf
cp /tmp/prova/jdg.keystore.jks ./jboss-eap-7.4
cp /tmp/prova/jdg.truststore.jks ./jboss-eap-7.4
```

# create JDG user
./redhat-datagrid-8.1.1-server/bin/cli.sh user create admin -p pass.1234

# configure JDG
Modify  to include the following:
```
    <server-identities>
        <ssl>
            <keystore path="jdg.keystore.jks" relative-to="infinispan.server.config.path"
                    keystore-password="123PIPPOBAUDO" alias="jdg-client" key-password="123PIPPOBAUDO"
                    />
        </ssl>
    </server-identities>
```

# start JDG
./redhat-datagrid-8.1.1-server/bin/server.sh -c infinispan.xml --cluster-stack=tcp --node-name=jdg1 

# configure WF: 
cat <<EOT > wf.cli
embed-server --server-config=standalone-ha.xml
/subsystem=jgroups/channel=ee:write-attribute(name=stack,value=tcp)
/subsystem=transactions:write-attribute(name=node-identifier,value=wildfly1)
/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=remote-jdg-server1:add(host=127.0.0.1, port=11222)
/subsystem=elytron/key-store=twoWayKS:add(path=jdg.keystore.jks,relative-to=jboss.home.dir,credential-reference={clear-text=123PIPPOBAUDO},type=JKS)
/subsystem=elytron/key-store=twoWayTS:add(path=jdg.truststore.jks,relative-to=jboss.home.dir,credential-reference={clear-text=123PIPPOBAUDO},type=JKS)
/subsystem=elytron/key-manager=twoWayKM:add(key-store=twoWayKS, algorithm="SunX509", credential-reference={clear-text=123PIPPOBAUDO})
/subsystem=elytron/trust-manager=twoWayTM:add(key-store=twoWayTS, algorithm="SunX509")
/subsystem=elytron/server-ssl-context=SERVER_SSL_CONTEXT:add(key-manager=twoWayKM, protocols=["TLSv1.2"], trust-manager=twoWayTM, need-client-auth=true)
batch
/subsystem=undertow/server=default-server/https-listener=https:undefine-attribute(name=security-realm)
/subsystem=undertow/server=default-server/https-listener=https:write-attribute(name=ssl-context, value=SERVER_SSL_CONTEXT)
run-batch
/subsystem=elytron/client-ssl-context=CLIENT_SSL_CONTEXT:add(key-manager=twoWayKM, trust-manager=twoWayTM, protocols=["TLSv1.2"])
batch
/subsystem=infinispan/remote-cache-container=session_data_cc:add(default-remote-cluster=jdg-server-cluster, module=org.wildfly.clustering.web.hotrod, protocol-version=3.0, statistics-enabled=true, properties={infinispan.client.hotrod.auth_username=admin, infinispan.client.hotrod.auth_password=pass.1234})
/subsystem=infinispan/remote-cache-container=session_data_cc/remote-cluster=jdg-server-cluster:add(socket-bindings=[remote-jdg-server1])
run-batch
/subsystem=infinispan/remote-cache-container=session_data_cc/component=security:write-attribute(name=ssl-context,value=CLIENT_SSL_CONTEXT)
/subsystem=distributable-web/hotrod-session-management=sm_offload:add(remote-cache-container=session_data_cc, granularity=SESSION)
/subsystem=distributable-web/hotrod-session-management=sm_offload/affinity=local:add()
/subsystem=distributable-web/hotrod-session-management=sm_offload_granular:add(remote-cache-container=session_data_cc, granularity=ATTRIBUTE)
/subsystem=distributable-web/hotrod-session-management=sm_offload_granular/affinity=local:add()
/subsystem=distributable-web:write-attribute(name=default-session-management,value=sm_offload)
/subsystem=infinispan/remote-cache-container=session_data_cc/component=security:write-attribute(name=ssl-context,value=CLIENT_SSL_CONTEXT)
EOT

./jboss-eap-7.4/bin/jboss-cli.sh --file=wf.cli

# start WF
./jboss-eap-7.4/bin/standalone.sh --server-config=standalone-ha.xml -Djboss.default.jgroups.stack=tcp -Dprogram.name=wfl1 -Djboss.node.name=wfl1

# deploy to WF
cp ../clusterbench-ee8.ear ./jboss-eap-7.4/standalone/deployments/

# TEST
curl http://localhost:8080/clusterbench/session
