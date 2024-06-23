# install-mongo-on-vps
Tutorial Install Mongodb

# Step-By-Step
**1. Update System Package –** <br>
```bash
sudo apt update
sudo apt upgrade
```
**2. Import MongoDB GPG Key  –** <br>
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
```
**3. Add MongoDB Repository -** <br>
```bash
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
```
***4. Reupdate your package system** <br>
```bash
sudo apt update
```
**5. Install MongoDB**<br>
```bash
sudo apt install -y mongodb-org
```
**6. Start MongoDB Services**<br>
```bash
sudo systemctl start mongod
```
**7. Idk but is important**<br>
```bash
sudo systemctl enable mongod
```
**8. Verify MongoDB Version**<br>
```bash
mongod --version
```


# Add Authentication
This Step is optional but for production env i just read this shit

**1. Logion to your mongodb shell**
```bash
mongo
```
**2. Create admin Database and add administrator users**
```javascript
use admin
db.createUser({
  user: "admin",
  pwd: "password123", // Ganti dengan password yang kuat
  roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
})
```
**3. Logout from mongodb shell**
```javascript
quit();
```

**4. Edit the MongoDB configuration file to enable authentication** <br>
Open the MongoDB configuration file:
```bash
sudo nano /etc/mongod.conf
```
by default mongodb config will look like this
```conf
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1


# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo

#security:

#operationProfiling:

#replication:
```
you can change bindIp to `0.0.0.0` to acccess mongodb via public ip. <br>

so back to the topic, we will add authentication on this shit <br>
you can delete # on security and add this on the next line
```bash
security:
  authorization: "enabled"
```

so the final config is
```bash
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0


# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo

security:
  authorization: "enabled"
```

**5. Restart MongoDB Services**
```bash
sudo systemctl restart mongod
```


# Authentication Test
**1. Try connection without authentication (should fail):** <br>
```bash
mongo
```
try this command
```bash
use admin
show dbs
```
if you get error msg or dbs not showing, than is good, walla your mongodb security shit is upgrade. Techlead smile++
**2. Try connection with authentication ** <br>
```bash
mongo -u "admin" -p "password123" --authenticationDatabase "admin"
```
and try this command
```bash
use admin
show dbs
```
if you not get dbs list, then your configuration is failed. your techlead will fuck your head<br>

thanks

# Fix Error
if you add privilage for admin user then u see this error
```bash
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Sun 2024-06-23 13:15:31 UTC; 6s ago
       Docs: https://docs.mongodb.org/manual
    Process: 6109 ExecStart=/usr/bin/mongod --config /etc/mongod.conf (code=exited, status=14)
   Main PID: 6109 (code=exited, status=14)

Jun 23 13:15:31 mongodb systemd[1]: Started MongoDB Database Server.
Jun 23 13:15:31 mongodb mongod[6109]: {"t":{"$date":"2024-06-23T13:15:31.805Z"},"s":"I",  "c":"CONTROL",  "id":7484500, "ctx":"main","msg":"Environment variable MONGODB_CONFIG_OVERRIDE_NOFORK == 1, overriding \"processManagement.fork\" to false"}
Jun 23 13:15:31 mongodb systemd[1]: mongod.service: Main process exited, code=exited, status=14/n/a
Jun 23 13:15:31 mongodb systemd[1]: mongod.service: Failed with result 'exit-code'.
```

you can follow this step

  1. Check your mongodb log


     Check the MongoDB logs for more information about why the service failed to start. This log is usually located at /var/log/mongodb/mongod.log:
     ```bash
     sudo tail -n 50 /var/log/mongodb/mongod.log
     ```
     if your mongodb log look like this
     ```json
      {"t":{"$date":"2024-06-23T13:11:43.207+00:00"},"s":"I",  "c":"CONTROL",  "id":23403,   "ctx":"initandlisten","msg":"Build Info","attr":{"buildInfo":{"version":"4.4.29","gitVersion":"f4dda329a99811c707eb06d05ad023599f9be263","openSSLVersion":"OpenSSL 1.1.1f  31           Mar 2020","modules":[],"allocator":"tcmalloc","environment":{"distmod":"ubuntu2004","distarch":"x86_64","target_arch":"x86_64"}}}}
      {"t":{"$date":"2024-06-23T13:11:43.207+00:00"},"s":"I",  "c":"CONTROL",  "id":51765,   "ctx":"initandlisten","msg":"Operating System","attr":{"os":{"name":"Ubuntu","version":"20.04"}}}
      {"t":{"$date":"2024-06-23T13:11:43.207+00:00"},"s":"I",  "c":"CONTROL",  "id":21951,   "ctx":"initandlisten","msg":"Options set by command line","attr":{"options":{"config":"/etc/mongod.conf","net":{"bindIp":"0.0.0.0","port":27017},"processManagement":                  {"timeZoneInfo":"/usr/share/zoneinfo"},"security":{"authorization":"enabled"},"storage":{"dbPath":"/var/lib/mongodb","journal":{"enabled":true}},"systemLog":{"destination":"file","logAppend":true,"path":"/var/log/mongodb/mongod.log"}}}}
      {"t":{"$date":"2024-06-23T13:11:43.207+00:00"},"s":"E",  "c":"NETWORK",  "id":23024,   "ctx":"initandlisten","msg":"Failed to unlink socket file","attr":{"path":"/tmp/mongodb-27017.sock","error":"Operation not permitted"}}
      {"t":{"$date":"2024-06-23T13:11:43.207+00:00"},"s":"F",  "c":"-",        "id":23091,   "ctx":"initandlisten","msg":"Fatal assertion","attr":{"msgid":40486,"file":"src/mongo/transport/transport_layer_asio.cpp","line":1048}}
      {"t":{"$date":"2024-06-23T13:11:43.207+00:00"},"s":"F",  "c":"-",        "id":23092,   "ctx":"initandlisten","msg":"\n\n***aborting after fassert() failure\n\n"}
      {"t":{"$date":"2024-06-23T13:15:31.805+00:00"},"s":"I",  "c":"CONTROL",  "id":20698,   "ctx":"main","msg":"***** SERVER RESTARTED *****"}
      {"t":{"$date":"2024-06-23T13:15:31.806+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"main","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}
      {"t":{"$date":"2024-06-23T13:15:31.811+00:00"},"s":"I",  "c":"NETWORK",  "id":4648601, "ctx":"main","msg":"Implicit TCP FastOpen unavailable. If TCP FastOpen is required, set tcpFastOpenServer, tcpFastOpenClient, and tcpFastOpenQueueSize."}
      {"t":{"$date":"2024-06-23T13:15:31.811+00:00"},"s":"I",  "c":"STORAGE",  "id":4615611, "ctx":"initandlisten","msg":"MongoDB starting","attr":{"pid":6109,"port":27017,"dbPath":"/var/lib/mongodb","architecture":"64-bit","host":"mongodb"}}
      {"t":{"$date":"2024-06-23T13:15:31.811+00:00"},"s":"I",  "c":"CONTROL",  "id":23403,   "ctx":"initandlisten","msg":"Build Info","attr":{"buildInfo":{"version":"4.4.29","gitVersion":"f4dda329a99811c707eb06d05ad023599f9be263","openSSLVersion":"OpenSSL 1.1.1f  31           Mar 2020","modules":[],"allocator":"tcmalloc","environment":{"distmod":"ubuntu2004","distarch":"x86_64","target_arch":"x86_64"}}}}
      {"t":{"$date":"2024-06-23T13:15:31.811+00:00"},"s":"I",  "c":"CONTROL",  "id":51765,   "ctx":"initandlisten","msg":"Operating System","attr":{"os":{"name":"Ubuntu","version":"20.04"}}}
      {"t":{"$date":"2024-06-23T13:15:31.811+00:00"},"s":"I",  "c":"CONTROL",  "id":21951,   "ctx":"initandlisten","msg":"Options set by command line","attr":{"options":{"config":"/etc/mongod.conf","net":{"bindIp":"0.0.0.0","port":27017},"processManagement":          {"timeZoneInfo":"/usr/share/zoneinfo"},"security":{"authorization":"enabled"},"storage":{"dbPath":"/var/lib/mongodb","journal":{"enabled":true}},"systemLog":{"destination":"file","logAppend":true,"path":"/var/log/mongodb/mongod.log"}}}}
      {"t":{"$date":"2024-06-23T13:15:31.811+00:00"},"s":"E",  "c":"NETWORK",  "id":23024,   "ctx":"initandlisten","msg":"Failed to unlink socket file","attr":{"path":"/tmp/mongodb-27017.sock","error":"Operation not permitted"}}
      {"t":{"$date":"2024-06-23T13:15:31.811+00:00"},"s":"F",  "c":"-",        "id":23091,   "ctx":"initandlisten","msg":"Fatal assertion","attr":{"msgid":40486,"file":"src/mongo/transport/transport_layer_asio.cpp","line":1048}}
      {"t":{"$date":"2024-06-23T13:15:31.811+00:00"},"s":"F",  "c":"-",        "id":23092,   "ctx":"initandlisten","msg":"\n\n***aborting after fassert() failure\n\n"}
        
     ```
     you can follow the next step
     
  3. Check Configuration

     Make sure the MongoDB configuration file (/etc/mongod.conf) has no errors. You can check the configuration file with a text editor like nano or vim:
     ```bash
     sudo nano /etc/mongod.conf
     ```
     
     Make sure that all configurations in the file are valid and appropriate. In particular, check the storage and systemLog sections to ensure the directories used actually exist and are accessible to MongoDB users.


     
  4. Check Directory and File Permissions
     Make sure the MongoDB data directory has proper permissions and is owned by the mongodb user:
     ```bash
     sudo chown -R mongodb:mongodb /var/lib/mongodb
     sudo chown -R mongodb:mongodb /var/log/mongodb
     ```
     
  5. Delete Socket Files Manually


     ```bash
     sudo rm /tmp/mongodb-27017.sock
     ```

  6. Check `/tmp` Directory Permissions

     Make sure the /tmp directory has the correct permissions. The /tmp directory must be accessible to all users:
     ```bash
     sudo chmod 1777 /tmp
     ```

  7. Restart MongoDB Service

      ```bash
      sudo systemctl start mongod
      ```
