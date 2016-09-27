<h3>HBase Snapshot to Spark Example</h3>

This project shows how to analyze an HBase Snapshot using Spark. 
<br>
<br>
<b>Why do this?</b>
<br>
The main motivation for writing this code is to reduce the impact on the HBase Region Servers while analyzing HBase records. By creating a snapshot of the HBase table, we can run Spark jobs against the snapshot, eliminating the impact to region servers and reducing the risk to operational systems.
<br>
<br><b>At a high-level, here's what the code is doing:</b>
  1. Reads an HBase Snapshot into a Spark
  2. Parses the HBase KeyValue to a Spark Dataframe
  3. Applies arbitrary data processing (timestamp and rowkey filtering)
  4. Saves the results back to an HBase (HFiles / KeyValue) format within HDFS, using HFileOutputFormat.
       - The output format maintains the original rowkey, timestamp, column family, qualifier, and value structure.
  5. From here, you can bulkload the HDFS file into HBase.

<br>
<br><b>Here's more detail on how to run this project:</b>
<br>
  1. Create an HBase table and populate it with data (or you can use an existing table). I've included two ways to simulate the HBase table within this repo (for testing purposes). Use the <a href="https://github.com/zaratsian/SparkHBaseExample/blob/master/src/main/scala/com/github/zaratsian/SparkHBase/SimulateAndBulkLoadHBaseData.scala">SimulateAndBulkLoadHBaseData.scala</a> code (preferred method) or you can use <a href="https://github.com/zaratsian/SparkHBaseExample/blob/master/write_to_hbase.py">write_to_hbase.py</a> (this is very slow compared to the scala code).
<br>
<br>
  2. Take an HBase Snapshot: <code>snapshot 'hbase_simulated_1m', 'hbase_simulated_1m_ss'</code>
<br>
<br>
  3. (Optional) The HBase Snapshot will already be in HDFS (at /apps/hbase/data), but you can use this if you want to load the HBase Snapshot to an HDFS location of your choice: ```hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot hbase_simulated_1m_ss -copy-to /tmp/ -mappers 2```
<br>
<br>
  4. Run the included Spark (scala) <a href="https://github.com/zaratsian/SparkHBaseExample/blob/master/src/main/scala/com/github/zaratsian/SparkHBase/SparkReadHBaseSnapshot.scala">code</a> against the HBase Snapshot. This code will read the HBase snapshot, process records, and output the data in an HBase (HFile/KeyValue) format.
<br>
<br>
      a.) Build project: ```mvn clean package```
<br>
<br>
      b.) Run Spark job: ```spark-submit --class com.github.zaratsian.SparkHBase.SparkReadHBaseSnapshot --jars /tmp/SparkHBaseExample-0.0.1-SNAPSHOT.jar /usr/hdp/current/phoenix-client/phoenix-client.jar /tmp/props```
<br>
<br>
      c.) NOTE: Adjust the properties within the props file (if needed) to match your configuration.

<br>
The Spark job will read the HBase Snapshot, filter records based on rowkey range (80001 to 90000) and based on a timestamp threshold (which is set in the props file), then write the results back to HDFS in HBase format (HFiles/KeyValue).
<br>
<br>
<b>Sample output of HBase simulated data structure (using SimulateAndBulkLoadHBaseData.scala):</b>
<img src="screenshots/Screen Shot 2016-09-27 at 10.58.13 AM.png" class="inline"/>
<br>
<br>
<b>Sample output of HBase simulated data structure (using write_to_hbase.py):</b>
<img src="screenshots/1_create_hbase_table.png" class="inline"/>
<br>
<br>
<br><b>Versions:</b>
<br>This code was tested using <a href="http://hortonworks.com/products/data-center/hdp/">Hortonworks HDP</a> 2.4.2.0-258 
<br>HBase version 1.1.2.2.4.2.0-258
<br>Spark version 1.6.1
<br>Scala version 2.10.5 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_40) 
