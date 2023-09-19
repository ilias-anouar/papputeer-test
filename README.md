# setuptourist

dependencies:
- node v13
- npm
- mysql

To establish a connection between your Node.js application running on your local machine and a PostgreSQL database on a CentOS 7 virtual machine, you need to follow these steps:

1. Configure PostgreSQL on CentOS 7:
   - Make sure PostgreSQL is installed and running on your CentOS 7 virtual machine.
   - Edit the PostgreSQL configuration to allow connections from remote IP addresses:
     - Locate the PostgreSQL configuration file, usually found at `/etc/postgresql/{version}/main/postgresql.conf`. Replace `{version}` with your PostgreSQL version.
     - Edit the `listen_addresses` parameter to accept connections from your local IP. You can set it to '*', which allows connections from any IP address. Change the line to look like this:
       ```
       listen_addresses = '*'
       ```
     - Save the file and restart PostgreSQL to apply the changes:
       ```
       sudo systemctl restart postgresql
       ```

2. Configure PostgreSQL to allow remote connections for your user:
   - Edit the `pg_hba.conf` file to specify which IPs are allowed to connect to PostgreSQL and which authentication method to use.
     - Locate the `pg_hba.conf` file, usually found at `/etc/postgresql/{version}/main/pg_hba.conf`.
     - Add the following line at the end of the file to allow connections from your local IP (127.0.0.1) using the "md5" authentication method:
       ```
       host    all    all    127.0.0.1/32    md5
       ```
     - Save the file.

3. Create a PostgreSQL user and database:
   - Log in to your CentOS 7 virtual machine.
   - Access the PostgreSQL command line using the `psql` command:
     ```
     sudo -u postgres psql
     ```
   - Create a new database and user with appropriate privileges. Replace `yourdatabase`, `youruser`, and `yourpassword` with your desired database name, username, and password:
     ```sql
     CREATE DATABASE yourdatabase;
     CREATE USER youruser WITH PASSWORD 'yourpassword';
     ALTER ROLE youruser SET client_encoding TO 'utf8';
     ALTER ROLE youruser SET default_transaction_isolation TO 'read committed';
     ALTER ROLE youruser SET timezone TO 'UTC';
     GRANT ALL PRIVILEGES ON DATABASE yourdatabase TO youruser;
     ```
   - Exit the PostgreSQL shell:
     ```
     \q
     ```

4. Install the PostgreSQL driver for Node.js in your local project:
   - In your Node.js project directory, run the following command to install the `pg` package, which is a PostgreSQL client for Node.js:
     ```
     npm install pg
     ```

5. Create a Node.js application to connect to the PostgreSQL database:
   - Create a JavaScript file (e.g., `app.js`) in your local project directory and add the following code to establish a connection to the PostgreSQL database:
     ```javascript
     const { Pool } = require('pg');

     // Replace with your PostgreSQL database connection details
     const pool = new Pool({
       user: 'youruser',
       host: 'your_centos_vm_ip', // Use the IP of your CentOS 7 virtual machine
       database: 'yourdatabase',
       password: 'yourpassword',
       port: 5432,
     });

     pool.query('SELECT NOW()', (err, res) => {
       if (err) {
         console.error('Error connecting to PostgreSQL:', err);
       } else {
         console.log('Connected to PostgreSQL:', res.rows[0].now);
       }
       pool.end();
     });
     ```

6. Run your Node.js application:
   - Execute your Node.js application with the following command in your project directory:
     ```
     node app.js
     ```

Your Node.js application should now be able to connect to the PostgreSQL database on your CentOS 7 virtual machine using the IP address of your virtual machine and the configured database credentials. Make sure your CentOS 7 virtual machine's firewall allows incoming connections on port 5432, or adjust your firewall settings as needed.



#error

If you're still encountering connection issues after configuring PostgreSQL to allow remote connections through Webmin, here are some troubleshooting steps to help you diagnose and resolve the problem:

1. **Check PostgreSQL Status**: Ensure that PostgreSQL is running on your CentOS 7 virtual machine. You can check its status using the following command:

   ```
   sudo systemctl status postgresql
   ```

   If it's not running, start it with:

   ```
   sudo systemctl start postgresql
   ```

2. **Firewall Configuration**: Make sure that your CentOS 7 firewall allows incoming connections on port 5432 (the default PostgreSQL port). You can use Webmin or the following command to open the port:

   ```
   sudo firewall-cmd --zone=public --add-port=5432/tcp --permanent
   sudo firewall-cmd --reload
   ```

3. **Check PostgreSQL Configuration in Webmin**: Revisit the PostgreSQL configuration in Webmin to ensure that you made the necessary changes to allow remote connections. Double-check the "Listen Addresses" and "Access Control" settings.

4. **Verify Connection Parameters in Node.js**: In your Node.js application code, verify that you're using the correct PostgreSQL host (the IP address of your CentOS 7 virtual machine), database name, username, and password. Also, ensure that you're using port 5432, which is the default PostgreSQL port.

5. **Test the Connection**: Try to connect to your PostgreSQL database from the CentOS 7 virtual machine itself using the `psql` command to confirm that PostgreSQL is accessible:

   ```
   psql -h your_centos_vm_ip -U youruser -d yourdatabase
   ```

   Replace `your_centos_vm_ip`, `youruser`, and `yourdatabase` with the appropriate values.

6. **Check for PostgreSQL Errors**: Review the PostgreSQL logs on your CentOS 7 virtual machine for any error messages that might provide clues about the connection issue. You can typically find PostgreSQL logs in the `/var/log/postgresql/` directory.

7. **Node.js Application Logging**: Add some logging statements to your Node.js application to capture any error messages or debug information related to the database connection.

8. **Network Connectivity**: Ensure that there are no network issues between your local machine and the CentOS 7 virtual machine. You can test network connectivity using tools like `ping` or `telnet` to verify that you can reach the virtual machine's IP address on port 5432.

9. **Security Groups or Network ACLs (If Using a Cloud VM)**: If you are running your CentOS 7 virtual machine on a cloud provider (e.g., AWS, Azure), check the security groups or network ACLs to ensure that they allow incoming traffic on port 5432 from your local IP address.

10. **Firewall on Your Local Machine**: Check if there's a firewall running on your local machine that might be blocking outgoing connections to port 5432.

By carefully reviewing and testing each of these steps, you should be able to identify and resolve the connection issue between your Node.js application and the PostgreSQL database on your CentOS 7 virtual machine.


## the code error generation :
-Error connecting to PostgreSQL: Error: connect ECONNREFUSED 127.0.0.1:5432
PS C:\Users\IT Team\Desktop\Report\Code source\server> node db.js
Error connecting to PostgreSQL: Error: connect ECONNREFUSED 127.0.0.1:5432
    at TCPConnectWrap.afterConnect [as oncomplete] (node:net:1495:16) {
  errno: -4078,
  code: 'ECONNREFUSED',
  syscall: 'connect',
  address: '127.0.0.1',
  port: 5432
}

## the function : 
const { Pool } = require('pg');

// Replace with your PostgreSQL database connection details
const pool = new Pool({
    user: 'root',
    host: '127.0.0.1', // Use the IP of your CentOS 7 virtual machine
    database: 'report',
    password: 'ilias080701',
    port: 5432,
});

pool.query('SELECT NOW()', (err, res) => {
    if (err) {
        console.error('Error connecting to PostgreSQL:', err);
    } else {
        console.log('Connected to PostgreSQL:', res.rows[0].now);
    }
    pool.end();
});


