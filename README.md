# Working and logging from an SQL server
Basic setup and testing of SQL server in Rocky 9

## Requirements:
1. Windows host (presumed Win 11).
2. Hypervisor (presumed VMWare).
3. A working installation of Splunk (see step 1 for assistance to install if required).
4. A recent Kali Linux VM.
5. The lab document.

## Step 1: Prepare a Rock 9 VM
You will need to install a minimal version of Rocky 9 (If you have an existing VM, go to step 2).

1. Follow the instructions in [step 1 here](https://github.com/zebra-acrobatic/splunk-botsv3) to build and configure a Rocky 9 VM.

## Step 2: Install SQL server

Follow these simple instructions to install an SQL Server on Rocky 9:

1. Install MySQL Server package:
```bash
sudo dnf install mysql-server
```

2. Start the MySQL service:
 ```bash
sudo systemctl start mysqld
```
3. Enable MySQL to start on boot:
```bash
sudo systemctl enable mysqld
```
4. Confirm the server is running as expected:
```bash
sudo systemctl status mysqld
```
5. Confirm the service is running by accessing it locally:
```bash
mysql -u root
```
6. If it works, use `exit` to quit the SQL shell.

7. Confirm the service is binded to the correct network port for SQL:
```bash
ss -tulpna | grep sql
```
8. Open the required firewall ports to allow the SQL server to communicate with clients.
```bash
firewall-cmd --add-port=3306/tcp --perm
```
9. Reload the firewall rules:
``` bash
firewall-cmd --reload
```

## Step 3: Configure basic security on the SQL server
The server will now be installed and accepting connections from the network. At this stage, it is very insecure and must be set up with some basic security.

1. Confirm the service is binded to the correct network port for SQL:
```bash
mysql_secure_installation
```
Follow the below prompts ensuring you use a root password you will not forget (`P@ssw0rd1` for example).
```
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: y

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 2
Please set the password for root here.

New password: 

Re-enter new password: 

Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
```
## Step 4: Configure more logging.
The logging facilities in the SQL server are very in depth and useful. There are many log types including (but not limited to):
 - `Error Log`: The server maintains an error log file that records critical errors, warnings, and informational messages encountered during server startup, shutdown, and regular operation. This log provides valuable insights into issues such as startup failures, syntax errors, and resource allocation problems.

 - `General Query Log`: The server can be configured to log all SQL statements executed by clients, including queries, updates, and administrative commands. The general query log facilitates performance analysis, query optimisation, and auditing of database activities. However, enabling this log can impact server performance due to the overhead of writing every query to the log file.

 - `Slow Query Log`: The server offers a slow query log that records SQL statements exceeding a specified execution time threshold. This log helps identify inefficient queries causing performance bottlenecks in the database. By analysing slow queries, database administrators can optimise query execution plans and improve overall system performance.

 - `Audit Plugin`: The server offers an Audit Plugin that enables fine-grained auditing of database activities. The Audit Plugin captures detailed information about user connections, authentication events, data access, and administrative operations. It helps organisations meet regulatory compliance requirements and detect unauthorised access or suspicious behavior within the database.

Further documentation is available [here](https://dev.mysql.com/doc/refman/8.3/en/server-logs.html). For now, you will need to activate some of these logs.

1. Verify what the server logs by default (notice, the logs will be sparse, lacking information.):
```bash
cat /var/log/mysql/mysqld.log
```
2. Check the logging settings for the SQL server (this will reveal which logs are on/off and where the log files will be):
```bash
mysql -uroot -p -se "SHOW VARIABLES" | grep -e log_error -e general_log -e slow_query_log
```
3. Turn on the general log with the following steps:
    - Open the configuration file using any text editor installed (`nano`,`vi` or `vim`), this example will use `vi`:
      ```bash
      vi /etc/my.cnf.d/mysql-server.cnf
      ```
    - Under the `[mysql]` block, add the line `general_log=1`.
    - Restart the server:
      ```bash
      systemctl restart mysqld
      ```
4. Check the logging settings for the SQL server have changed:
```bash
mysql -uroot -p -se "SHOW VARIABLES" | grep -e log_error -e general_log -e slow_query_log
```
## Step 5: Connect to the server remotely
In reality, database administration is rarely done on the server itself. You will now use some simple tools to connect to and manage the SQL server remotely.

1. The `mysql_secure_installation` script disabled remote root access, you will need to create another administrative user, start by logging into the SQL server locally.
```bash
mysql -u root -p
```
2. Add a new administrative user (use your first name as the username).
```sql
CREATE USER 'yourname'@'localhost' IDENTIFIED BY 'P@ssw0rd1';
```
3. Add permissions to the new user:
```sql
GRANT ALL PRIVILEGES ON *.* TO 'yourname'@'localhost' WITH GRANT OPTION;
```
4. Create and grant perssmisions for the same user to log in remotely:
```sql
CREATE USER 'yourname'@'%' IDENTIFIED BY 'P@ssw0rd1';
```
```sql
GRANT ALL PRIVILEGES ON *.* TO 'yourname'@'%' WITH GRANT OPTION;
```
5. On your Windows host, go to the [Heidisql download page.](https://www.heidisql.com/download.php)
6. Download the portable version.
7. Extract all the files in the zip.
8. Run `heidisql.exe`
9. Set the following options in HeidiSQL:
 - `Hostname/IP`: Put the IP address of your SQL server
 - `Prompt for credentials`: Tick
 -  Press `Save`
10. Press `Connect` and enter the new username and password when requested.

## Step 6: Create a new database
Use your remote connection tool to create and manage a new simulated customer database. Pay attention to the bottom section of the HeidiSQL GUI, it will provide the SQL language syntax needed to answer questions in the lab.

1. In HeidiSQL, right click `Unnamed` and click `Create New` > `Database`. Name the database `customers`.
2. Right click the new Database and create a new table with the following properties:

| Column 	| Name       	| Data type 	| Comment       	|
|--------	|------------	|-----------	|---------------	|
| 1      	| cust_id    	| INT       	| Customer ID   	|
| 2      	| first_name 	| VARCHAR   	| First name    	|
| 3      	| last_name  	| VARCHAR   	| Last Name     	|
| 4      	| email      	| VARCHAR   	| Email address 	|
| 5      	| ph         	| VARCHAR   	| Phone Number  	|

sd
