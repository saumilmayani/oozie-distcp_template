# oozie-distcp_template

Writing an Oozie Distcp Action: 
In a Kerberized environment, how to run Oozie workflow with two HDFS HA clusters without updating properties on server ? 

Steps:

1. Update job.properties file

* Reflect Namenode HA and YARN Cluster-id related information from 2 Clusters.
Following files from 2 clusters would be needed.

```
/etc/hadoop/conf/core-site.xml
/etc/hadoop/conf/yarn-site.xml
/etc/hadoop/conf/hdfs-site.xml
```

* Following commands would help you source this information out.

```
grep -A1 'fs.defaultFS' /etc/hadoop/conf/*
grep -A1 'yarn.resourcemanager.cluster-id' /etc/hadoop/conf/*
grep -A1 'dfs.nameservices' /etc/hadoop/conf/*
grep -A1 'dfs.internal.nameservices' /etc/hadoop/conf/*
grep -A1 'dfs.client.failover.proxy.provider' /etc/hadoop/conf/*
grep -A1 'dfs.ha.namenodes' /etc/hadoop/conf/*
grep -A1 'dfs.namenode.rpc-address' /etc/hadoop/conf/*
```

```
Example:
# grep -A1 'fs.defaultFS' /etc/hadoop/conf/*

nameNode=hdfs://shmpsehdpenv1

# Cluster where the job will be initiated
# grep -A1 'yarn.resourcemanager.cluster-id' /etc/hadoop/conf/*

jobTracker=yarn-cluster

# YARN QUEUE 

property01=mapred.job.queue.name
value01=default

# 12 Cluster specifics properties 

# comma separated list of file system names, from both engi
# grep -A1 'fs.defaultFS' /etc/hadoop/conf/*

property02=mapreduce.job.hdfs-servers
value02=hdfs://shmpsehdpenv1,hdfs://smayanihdp

# Comma separated list of nameservices
# grep -A1 'dfs.nameservices' /etc/hadoop/conf/*

property03=dfs.nameservices
value03=shmpsehdpenv1,smayanihdp

# Only include the cluster where you intend to run the job.
# grep -A1 'dfs.internal.nameservices' /etc/hadoop/conf/*

property04=dfs.internal.nameservices
value04=shmpsehdpenv1

# grep -A1 'dfs.client.failover.proxy.provider' /etc/hadoop/conf/*

property05=dfs.client.failover.proxy.provider.shmpsehdpenv1
value05=org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider

# grep -A1 'dfs.ha.namenodes' /etc/hadoop/conf/*

property06=dfs.ha.namenodes.shmpsehdpenv1
value06=nn1,nn2

# grep -A1 'dfs.namenode.rpc-address' /etc/hadoop/conf/*

property07=dfs.namenode.rpc-address.shmpsehdpenv1.nn1
value07=shmpsehdpenv1n2.hortonworks.com:8020

property08=dfs.namenode.rpc-address.shmpsehdpenv1.nn2
value08=shmpsehdpenv1n3.hortonworks.com:8020

# grep -A1 'dfs.client.failover.proxy.provider' /etc/hadoop/conf/*

property09=dfs.client.failover.proxy.provider.smayanihdp
value09=org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider

# grep -A1 'dfs.ha.namenodes' /etc/hadoop/conf/*

property10=dfs.ha.namenodes.smayanihdp
value10=nn1,nn2

# grep -A1 'dfs.namenode.rpc-address' /etc/hadoop/conf/*

property11=dfs.namenode.rpc-address.smayanihdp.nn1
value11=hdpnode2.openstacklocal:8020

property12=dfs.namenode.rpc-address.smayanihdp.nn2
value12=hdpnode3.openstacklocal:8020

# RM uses these configurations to renew tokens

property13=mapreduce.job.send-token-conf
value13=mapreduce.jobhistory.principal|^dfs.nameservices|^dfs.namenode.rpc-address.*|^dfs.ha.namenodes.*|^dfs.client.failover.proxy.provider.*|dfs.namenode.kerberos.principal
```

* Update the workflow directory

```
examplesRoot=oozie-distcp-workflow
oozie.wf.application.path=${nameNode}/user/${user.name}/${examplesRoot}/workflow.xml
```

* Update the HDFS Source and Destination path

```
source=hdfs://smayanihdp/user/user01t01/passwd
destination=hdfs://shmpsehdpenv1/user/user01t01/passwd
```

2. Execute Oozie Workflow.

```
kinit user01t01

hdfs dfs -rm -r -skipTrash oozie-distcp-workflow
 
hdfs dfs -put oozie-distcp-workflow
source /usr/hdp/current/oozie-client/conf/oozie-env.sh ; 

/usr/hdp/current/oozie-client/bin/oozie job -config job.properties -run

/usr/hdp/current/oozie-client/bin/oozie job -info 0000001-180227164558355-oozie-oozi-W -verbose
```
