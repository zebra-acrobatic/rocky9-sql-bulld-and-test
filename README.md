# rocky9-sql-bulld-and-test
Basic setup and testing of SQL server in Rocky 9

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
