# Streaming Telemetry 101

In this tutorial you will learn how to configure streaming telemetry for XE, NX & XR, use collectors, transform the data for a time series database, and visualize the data in Grafana.

## The Environment

In this environment there are three switches/routers, CSR1000v, NX9kv, and XRv, connected to Ubuntu server. The Ubuntu server has 5 main components:

1. Consumers (Netconf Client & [Pipeline](https://github.com/cisco/bigmuddy-network-telemetry-pipeline))
2. [Kafka](https://kafka.apache.org/) (Message Bus)
3. [Apache Spark](https://spark.apache.org/streaming/) w/ [Jupyter](http://jupyter.org/) (Transform the data into the time-series database format)
4. [Opentsdb](http://opentsdb.net/) (Time-series database)
5. [Grafana](https://grafana.com/) (Visualization)

![environment](environment.png)



## Start the databases

For this setup, we are using Apache Spark as the transformer and OpenTSDB as the time series database. 

1. SSH into the Server

   `ssh telemetry@10.10.20.25` password is `Cisco1234!`

2. Start the docker stacks

   ```
   cd telemetry_stacks
   docker-compose -f docker-compose.yml -f docker-compose-opentsdb.yml up -d
   ```

   

## Setting up Telemetry on XE

Streming telemetry on XE uses NETCONF streaming. NETCONF Streaming is a feature added by Cisco to NETCONF in their custom Python client, [ncc](https://github.com/CiscoDevNet/ncc). Due to the fact they use NETCONF, only XML is supported in encoding.  

**Ensure that you start the databases first

#### Configure Telemetry

1. First, we will need to turn on streaming telemetry on XE. Start by ssh into the box. The password is `Cisco1234!`.

   `ssh admin@10.10.20.30`

2. Next turn on netconf-yang and netconf to allow streaming telemetry.

   ```
   CSR1000V#conf t
   Enter configuration commands, one per line.  End with CNTL/Z.
   CSR1000V(config)#netconf-yang
   CSR1000V(config)#exit
   CSR1000V#
   ```

#### Turn on consumer

1. SSH into the server (Password is `Cisco1234!`)

   `ssh telemetry@10.10.20.25`

2. Turn on the sample script to test that telemetry is working. To learn more about the sample script, check it out [here](dummy placeholder).

   ```shell
   cd telemetry_stacks/
   cd consumers/
   cd xe
   pipenv shell
   python print_telemetry.py
   ```

   If everything is working, you should see an telemetry output in xml similar to this:

   ```xml
   .
   .
   <datastore-contents-xml xmlns="urn:ietf:params:xml:ns:yang:ietf-yang-push">
     <interfaces-state xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
       <interface>
         <name>Control Plane</name>
         <statistics>
           <discontinuity-time>2018-05-31T14:49:17.000193+00:00</discontinuity-time>
           <in-octets>0</in-octets>
           <in-unicast-pkts>0</in-unicast-pkts>
           <in-broadcast-pkts>0</in-broadcast-pkts>
           <in-multicast-pkts>0</in-multicast-pkts>
           <in-discards>0</in-discards>
           <in-errors>0</in-errors>
           <in-unknown-protos>0</in-unknown-protos>
           <out-octets>0</out-octets>
           <out-unicast-pkts>0</out-unicast-pkts>
           <out-broadcast-pkts>0</out-broadcast-pkts>
           <out-multicast-pkts>0</out-multicast-pkts>
           <out-discards>0</out-discards>
           <out-errors>0</out-errors>
         </statistics>
       </interface>
   .
   .
   .
   ```

   

3. Now let's send the data to Kafka instead. Instead of running `print_telemetry.py`, let's run `kafka_telemetry.py` in the background. 

   ```shell
   python kafka_telemetry.py > telemetry.log 2>&1 &
   ```

   

#### Start the transformer

1. Navigate to `10.10.20:25:8888` in a browser, the password is `cisco1234`

2. Click on the folder `telemetry`

3. Click on the file [Kafka_OTSDB_XE.ipynb](http://10.10.20.25:8888/notebooks/telemetry/Kafka_OTSDB_XE.ipynb)

4. Click on the first tile, and click run on all the tiles but the last one that says `ssc.stop()`

   *** Make sure you do not click run on `ssc.stop`**

#### Create a graph

1. Navigate to `10.10.20.25:3000` in a browser, the login details is admin/admin

2. Click on Add Data Source in the center of the screen

3. Change the details to the following:

   ​	Name: OTSDB

   ​	Type: OpenTSDB

   ​	URL:  http://opentsdb:4242

4. Click on `Save & Test`, if everything is working you should see a green box saying "Data source is working"

5. Click on the "+" icon and create a new dashboard

6. Add a graph

7. Click on the panel and press "e"

8. Change the Metric to "in-octets" and Aggregator to "last" and check the "Rate" box

9. Click the refresh icon on the top right of the box, you should now see data
