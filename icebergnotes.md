What is a table format?

-	A way to organize a dataset’s files to present them as a single table. Our datalake may contain millions of data files that represent multiple datasets. The analytical tools need to know which files are part of which dataset. This is the problem that table formats solve.
-	It’s a way to answer the question “what data is in this table”?

Hive Table Format:

-	Old table format.
-	Abstracted the complexity of Map Reduce. Internally it converted SQL statements in to map reduce jobs. 
-	Created at Facebook
-	Used “directories” on storage (HDFS/Object Storage) to indicate which files are part of which table. This format applies a simpler approach that says a particular directory is a table. All the contents of the directory are part of this table. Any subfolders would be treated as partitions of the table.

-	Pros:
o	Works with almost every engine since it became the de-facto standard and stayed so for long.
o	Partitioning allows more efficient query access patterns than full table scans.
o	File format agnostic. 
o	Atomically update a while partitions. 
o	Single, central answer to “what data is in this table” for the whole ecosystem.

-	Cons:
o	Small updates are very inefficient. Updating a few records wasn’t easy. A partition was a smallest unit of transaction. To update few records, the entire partition or entire table had to be swapped/rewritten.
o	No way to atomically update multiple partitions. There is a risk that after one partition is changed and before we start work on the next partition, someone could query the table and read inconsistent data 
o	Multiple jobs modifying the same dataset do not do it safely. ,.i.e. Safe concurrent writes not possible.
o	All of the directory listing needed for large table take a long time. Engines have to list all the contents of the directories prior to returning the results so that it can understand the metadata of the files that make up the table/partition.
o	To filter out the files (pruning) required the engines to read those files. Opening and closing of all the files to check if they have the data of interest made it time consuming.
o	Users of the datasets have to know the physical layout of the table. Without knowing the physical layout of the table, we might write inefficient queries that could lead to full table scans that are costly.
o	Hive table statistics are often stale and data engineers have to keep executing Analyze queries to keep the stats up to date. Collecting stats needs more compute and stats are only fresh as often we run the analyze jobs.




Netflix tried to address the problems with the Hive table format, by creating a new table format.
The goals Netflix wanted to achieve with this format were:
-	Table correctness/consistency
-	Faster query planning and inexpensive execution – eliminate file listing and better file pruning
-	Allows users to not worry about the physical layout of the data. Users should be able to write queries naturally and be able to get benefit or partitioning.
-	Table evolution – Allow adding, removal of table columns, changes to partition schema.
-	Accomplish all of these at scale.

With this we landed with the concept of iceberg.
"With Iceberg, a table is a canonical list of files." 
Instead of saying a table is all the files in a directory, theoretically the files in iceberg table could be anywhere. Files could be in different folders and does not have to be in nicely organized directory system.
This is because iceberg is going to maintain the list of files in its metadata and the engine would leverage this metadata. This allows to get to the files faster.




What iceberg is?
1)	Table Format Specification:

It’s a standard for how do we organize the data around the table.
Any engine reading/writing data from iceberg table should implement this open specification.

2)	A set of APIs and libraries of interaction with that specification (Java, Python API)

These libraries are leveraged in other engines and tools that allow them to interact with iceberg tables.


What iceberg is not?
1)	Not a storage engine. (Storage options we could use are HDFS, object stores like S3)

2)	Not an execution engine (Engine options we could use Spark, Presto, Flink etc)

3)	Not a service. We do not have to run a server of some sort. It’s just a specification for storing and reading data and metadata files.






Iceberg Design Benefits:
-	Efficiently make smaller updates. Updates do not happen at the directory level now. Changes are made at the file level.
-	Snapshot isolation for transactions. Every change to the table creates a new snapshot. All read are on the newest snapshot. Reads and writes do not interfere with each other and all writes are atomic. Read does not read a partial snapshot.
-	Faster planning and execution. A lot of metadata is maintained at individual file, partition level allowing better file pruning while running the query. Column stats are maintained in the manifest files. These column stats are used to eliminate files.
-	Reliable metrics for CBOs(vs hive). The column stats and metrics are calculated on write instead of frequent expensive jobs.
-	Abstracting the physical, expose a logical view. Users don’t have to be familiar with the underlying physical structure of the table. Features like hidden partitioning, compaction of small files , table can change over time , ability to experiment with the table layout with breaking the table.
-	Rich schema evolution support.
-	All engines see changes immediately.
<img width="1710" alt="Screenshot 2025-06-07 at 12 15 08 PM" src="https://github.com/user-attachments/assets/71facbf7-fa2b-474b-9912-9013bbc57a4e" />
<img width="1710" alt="Screenshot 2025-06-07 at 12 20 36 PM" src="https://github.com/user-attachments/assets/2f8373da-b82c-4a68-8ec2-4a91ad089e68" />
<img width="1710" alt="Screenshot 2025-06-07 at 12 25 03 PM" src="https://github.com/user-attachments/assets/781f974a-93ab-4074-9439-40f7a69d2616" />
<img width="1710" alt="Screenshot 2025-06-07 at 12 28 42 PM" src="https://github.com/user-attachments/assets/e6e8c3b3-8a25-4735-9afb-450d8d4df8c1" />
<img width="1710" alt="Screenshot 2025-06-07 at 12 34 09 PM" src="https://github.com/user-attachments/assets/9d1e2d26-1127-454c-829d-61c2f10cbf6b" />
<img width="1710" alt="Screenshot 2025-06-07 at 12 35 21 PM" src="https://github.com/user-attachments/assets/3ae09079-904d-4095-b081-c90b3be43960" />
