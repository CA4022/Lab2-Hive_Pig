This lab covers material from week 3 and will span across the next 2 weeks.
We will install HIVE and PIG and start practicing with their basic functions separately. 
Both of these applications can either run on top of a working HDFS / Mapreduce cluster, or they can run in local mode (useful for debugging).

# Lab2 - HIVE
## Download and Install Hive
You can find an overall guide [here](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHive). Key steps are summarised below.
<!-- install on mac: https://bigdatalatte.wordpress.com/2017/02/01/install-hadoop-yarn-hive-on-a-macbook-pro-el-capitan/-->

* Get and install hive from one of the [mirror sites](http://www.apache.org/dyn/closer.cgi/hive/), copying it into the same directory where you have your hadoop root

   - `$ wget https://ftp.heanet.ie/mirrors/www.apache.org/dist/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz `
   - `$ tar -xvzf apache-hive-3.1.2-bin.tar.gz `
   - `$ cp -avi apache-hive-3.1.2-bin /usr/local/Cellar/hive-3.1.2`
   
* Note you will need Java >6 and Hadoop 0.20.x or greater  
* Make sure your $HADOOP_HOME environment variable is correctly set
* Set $HIVE_HOME and add hive to PATH
   
   - `$ export HIVE_HOME=/usr/local/Cellar/hive-3.1.2`
   - `$ export PATH=/usr/local/Cellar/hive-3.1.2/bin:$PATH`
   
* Bootstrap locations used to store files on HDFS (you need to be in your hive home directory)
   - `$ bin/hdfs dfs -mkdir /tmp` should already exist from hadoop installation
   - `$ bin/hdfs dfs -mkdir /user/hive/warehouse`
   - `$ bin/hdfs dfs -chmod g+w /tmp`should already have the right access rights from hadoop installation
   - `$ bin/hdfs dfs -chmod g+w /user/hive/warehouse`
   
* Configuration file (optional)
   - `$ cp $HIVE_HOME/conf/hive-default.xml.template $HIVE_HOME/conf/hive-site.xml`
   
* Initialise derby database (mysql also an option but derby is easier to use)
  - `$ $HIVE_HOME/bin/schematool –initSchema –dbType derby`
  - Note: you might need to check your hive-site.xml if you get errors on the type of database and make sure it is set to be derby
   
* Run the shell (of the three modes to run HIVE, we will use command line)
  - `$ $HIVE_HOME/bin/hive`

## Hive Examples
* Check examples to Create, Load, Query data with Hive:
  - [Simple example queries](https://www.java-success.com/10-setting-getting-started-hive-mac/)
  - [Hive Examples (slide 17 onwards)](http://courses.coreservlets.com/Course-Materials/pdf/hadoop/07-Hive-01.pdf)
  - [Import CSV file and generate database](http://djkooks.github.io/hadoop-hive-setup)
   
* Create your own tables and load data from a .txt file you have created or downloaded, then practice some queries.
* You can find more query examples and SQL cheat-sheet [here](https://hortonworks.com/blog/hive-cheat-sheet-for-sql-users/)
   
## Hive errors and fixes
Below is a list of possible errors you might encounter when installing and running Hive, and how to fix them.

*  [Guava incompatibility error](https://phoenixnap.com/kb/install-hive-on-ubuntu)
*  "Name node is in safe mode" error:
   - Check if namenode safemode is ON: `$ $HADOOP_HOME/bin/hadoop dfsadmin –safemode get`
   - If this is the case, disable it: `$ $HADOOP_HOME/bin/hadoop dfsadmin –safemode leave`
* URISyntaxException: Check the problematic string into hive-site.xml file and replace it with correct path

<!-- check this as well:https://kb.databricks.com/metastore/hive-metastore-troubleshooting.html-->

# Lab2 - PIG
## Download and Install PIG
The official apache manual is availalbe [here](http://pig.apache.org/docs/r0.17.0/start.html). Key steps are summarised below.

* Get and install pig from one of the [mirror sites](http://www.apache.org/dyn/closer.cgi/pig/), copying it into the same directory where you have your hadoop root

   - `$ wget https://ftp.heanet.ie/mirrors/www.apache.org/dist/pig/pig-0.17.0/pig-0.17.0.tar.gz `
   - `$ tar -xvzf pig-0.17.0.tar.gz`
   - `$ cp -avi pig-0.17.0.tar.gz /usr/local/Cellar/pig-0.17.0`
   
* Note you will need Java >6 and Hadoop 2.x or greater  
* Make sure your $HADOOP_HOME environment variable is correctly set
* Set $PIG_HOME and add pig to PATH
   
   - `$ export PIG_HOME=/usr/local/Cellar/pig-0.17.0`
   - `$ export PATH=$PATH:$PIG_HOME/bin`

* Test your installation: `$ pig -help`
* Run the grunt shell
  - in local mode: `$ pig -x local`
  - in mapreduce mode (default): `$ pig -x mapreduce`
* Exit the grunt shell: `$grunt> quit`

## PIG Examples
1. Let's try Wordcount (locally first)

```
lines = LOAD 'local_file.txt' AS (line:chararray);
words = FOREACH lines GENERATE FLATTEN(TOKENIZE(line)) as word;
grouped = GROUP words BY word;
wordcount = FOREACH grouped GENERATE group, COUNT(words);
DUMP wordcount;
```
* NOTE: if you run pig on mapreduce, you need to make sure the input file is on HDFS, using 'hdfs://localhost:8020/user/<username>/input/input_file.txt'

2. A Toy Example:
      - Create two CSV file as follows:
            * A.txt containing two lines: (0,1,2) and (1,3,4)
            * B.txt containing two lines: (0,5,2) and (1,7,8)
      - Run the PIG Latin commands below, one by one from shell, and observe what is contained in d, e, f and g after each dump (note the content of g and reflect upon the way pig handles schema on the fly
```
a = LOAD ‘A.txt’ using PigStorage(‘,’); 
b = LOAD ‘B.txt’ using PigStorage(‘,’); 
c = UNION a,b;
SPLIT c INTO d IF $0==0, e IF $0==1; 
dump d;
dump e;
f = FILTER c BY $1>3;
dump f;
A = LOAD ‘A.txt’ using PigStorage(‘,’) as (a1:int, a2:int, a3:int); 
B = LOAD ‘B.txt’ using PigStorage(‘,’) as (b1:int, b2:int, b3:int); C = UNION A,B;
g = FOREACH c GENERATE a2, a2*a3
dump g;
```

3. More examples:
   - [Slides from coreservlets](http://courses.coreservlets.com/Course-Materials/pdf/hadoop/06-Pig-01-Intro.pdf) 
   - [git script examples](https://gist.github.com/brikis98/1332818)


