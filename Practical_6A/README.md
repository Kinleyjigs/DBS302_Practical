# Practical 6A


## Learning Objectives

By the end of this practical, we should be able to enable authentication and configure ACL users in Redis with role-based permissions, generate self-signed TLS certificates for encrypted communication, and perform a basic security audit to verify that access controls are correctly enforced.

# Part A Securing Redis

## Step 0: Verify Redis Installation

```python
redis-server --version
redis-server &
redis-cli ping
```

![alt text](<images/image 1.png>) 

![alt text](<images/image 2.png>) 

![alt text](<images/image 3.png>)

*Figure 1: Redis Running* 

*Redis server started successfully and responded to PING with PONG, confirming it is running on macOS.*

## Step 1. Enable ACL and Create Users

### Using **redis.conf**

Finding the redis.conf file.

```python
find / -name "redis.conf" 2>/dev/null | head -5
```

Then, we will open the config file to edit it. 

```python
sudo nano /opt/homebrew/etc/redis.con
```

The following configuration block defines the Redis ACL users in redis.conf. The default user is disabled to block anonymous access, and three users admin, app_user, and monitoring are created, each with a unique password and strictly limited permissions.

![alt text](<images/image 4.png>)

Then restart and test:

```python
brew services restart redis
redis-cli -u redis://admin:adminStrongPwd@127.0.0.1:6379 ping
```

![alt text](<images/image 5.png>)

*Figure 2: ACL_Config_Restart*
Redis restarted with ACL configuration. Admin user authenticated successfully. PONG received despite security warning.

### Testing ACL Users with redis-cli

Now each user will be tested to verify authentication and ACL permissions. 

Test admin

```python
redis-cli -u redis://admin:adminStrongPwd@127.0.0.1:6379
```

![alt text](<images/image 6.png>)

*Figure 3:Admin_SetGet
Admin user connected successfully. SET and GET commands work as expected confirming full access. Note: whoami command not supported in Redis 8.x ACL WHOAMI is the correct replacement.*

Test app_user

```python
redis-cli -u redis://app_user:appStrongPwd@127.0.0.1:6379
```

![alt text](<images/image 7.png>)

*Figure 4: AppUser_RBAC_Test
AppUser was correctly able to set/get session:user123 within the appropriate key pattern of session and was denied access on otherkey.  Otherkey was restricted based on Redis ACL Key-level Permitted Access Pattern.*

test the monitoring user.

![alt text](<images/image 8.png>)

*Figure 5: Monitoring_ReadOnly
The Monitoring User was able to read session:user123 but was correctly blocked from writing, confirming the read-only access restriction on Redis ACL.*

## Step 2: Enabling TLS for Redis

### Generate Self-Signed Certificates

![alt text](<images/image 9.png>)

*Figure 6: TLS_Certificates_Generated*

*Here I used OpenSSL to create security certificates for Redis.
This includes the main certificate (CA) and the Redis server’s own certificate and key.
All the files were created successfully and saved in /etc/redis/tls/*

Let's verify all the security files are there:

```python
ls -la /etc/redis/tls/
```

![alt text](<images/image 10.png>)

*Figure 7: TLS_Files_List
All TLS certificate files are confirmed present in /etc/redis/tls/*

### Update redis.conf for TLS

The redis.conf file was updated with the following TLS configuration parameters as specified in the practical guide.

![alt text](<images/image 11.png>)

Restart Redis again:

![alt text](<images/image 12.png>)

### Connect Using TLS

![alt text](<images/image 13.png>)

*Figure 8: TLS_Config_Commented
Here i have commented out the TLS configuration lines in redis.conf because Redis 8.x on macOS Homebrew does not have built-in support for TLS. I generated the certificates using OpenSSL, which demonstrated the installation process. The plain port 6379 has also been restored to allow Redis to function properly.*

![alt text](<images/image 14.png>)

*Figure 9: Admin_TLS_Session_Test
The Admin user was successfully connected and was verified using the ACL WHOAMI command. A test key (session:user456) was also set and retrieved without issues, confirming that Redis ACL permissions and session key access are working properly after the TLS configuration.*

### Simple Application Code Demo for Redis (Python)




## Reflection

From this practical i have learned how the database security works in real situation. I learned how to use the Redis ACL to create users with different roles and restrict both commands and key access, which showed how flexible and detailed access control can be.

I also learned the importance of testing security properly not only checking what works, but also confirming that unauthorized actions are blocked. The TLS setup and troubleshooting taught me that security configurations depend on the system environment, and problems may occur even when steps are followed correctly.

Overall, this practical improved my understanding of authentication, role-based access control, and encryption, and showed me the importance of careful testing and environment awareness when implementing database security.