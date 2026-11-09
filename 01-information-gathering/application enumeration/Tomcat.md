**Overview:**
- Apache Tomcat is an open-source web server that hosts applications written in Java. Tomcat was initially designed to run Java Servlets and Java Server Pages (JSP) scripts. However, its popularity increased in Java-based frameworks and is now widely used by frameworks such as Spring and tools such as Gradle.
- Tomcat is often less apt to be exposed to the internet. It's seen from time to time on external pentests and can make for an excellent foothold into the internal network.
- It is far more common to see Tomcat (and multiple instances, for that matter) during internal pentests. More often than not, at least one instance internal is configured with weak or default credentials.
---
## Discovery

Tomcat can be identified via the `Server` header in the HTTP response. Behind a reverse proxy, requesting an invalid page may reveal the server and version.

---
## Directory Structure

```
├── bin                         # Scripts and binaries to start/run Tomcat
├── conf
│   ├── tomcat-users.xml        # User credentials and roles
│   └── web.xml
├── lib                         # JAR files required for Tomcat
├── logs                        # Log files
├── temp                        # Temporary files
├── webapps                     # Default web root, hosts all applications
│   ├── manager
│   └── ROOT
└── work                        # Runtime cache
```

Key file: `conf/tomcat-users.xml` stores credentials. `webapps/` hosts all deployed applications.

Each application follows this structure:

```
webapps/customapp
├── index.jsp
├── META-INF/context.xml
└── WEB-INF
    ├── web.xml                 # Deployment descriptor — routes and handler classes
    ├── jsp/admin.jsp
    └── lib/jdbc_drivers.jar
```

`WEB-INF/web.xml` is the most important file — it maps routes to their handler classes.

---
## Enumeration

```bash
gobuster dir -u http://<target>:<port>/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
```

Enumerate directories. Look for `/manager` and `/host-manager` . These are the primary attack targets.