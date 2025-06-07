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



<img width="1710" alt="Screenshot 2025-06-07 at 12 15 08 PM" src="https://github.com/user-attachments/assets/71facbf7-fa2b-474b-9912-9013bbc57a4e" />
<img width="1710" alt="Screenshot 2025-06-07 at 12 20 36 PM" src="https://github.com/user-attachments/assets/2f8373da-b82c-4a68-8ec2-4a91ad089e68" />
<img width="1710" alt="Screenshot 2025-06-07 at 12 25 03 PM" src="https://github.com/user-attachments/assets/781f974a-93ab-4074-9439-40f7a69d2616" />
<img width="1710" alt="Screenshot 2025-06-07 at 12 28 42 PM" src="https://github.com/user-attachments/assets/e6e8c3b3-8a25-4735-9afb-450d8d4df8c1" />
<img width="1710" alt="Screenshot 2025-06-07 at 12 34 09 PM" src="https://github.com/user-attachments/assets/9d1e2d26-1127-454c-829d-61c2f10cbf6b" />
<img width="1710" alt="Screenshot 2025-06-07 at 12 35 21 PM" src="https://github.com/user-attachments/assets/3ae09079-904d-4095-b081-c90b3be43960" />
