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
