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

* Update the workflow directory

* Update the HDFS Source and Destination path

2. Execute Oozie Workflow.

```
kinit user01t01

hdfs dfs -rm -r -skipTrash oozie-distcp-workflow
 
hdfs dfs -put oozie-distcp-workflow
source /usr/hdp/current/oozie-client/conf/oozie-env.sh ; 

/usr/hdp/current/oozie-client/bin/oozie job -config job.properties -run

/usr/hdp/current/oozie-client/bin/oozie job -info 0000001-180227164558355-oozie-oozi-W -verbose
```
