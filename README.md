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
