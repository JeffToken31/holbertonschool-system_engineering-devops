# Scale Up Web Infrastructure – Explanation

## How the request flows

```mermaid
---
config:
  layout: elk
---
flowchart BT
 subgraph HAProxyCluster["HAProxy Load Balancer Cluster (Active-Active)"]
        HAProxy1["HAProxy 1"]
        HAProxy2["HAProxy 2"]
  end
 subgraph WebServers["Web Servers"]
        FWWeb["Firewall"]
        SSLWeb["SSL Certificate"]
        NginxWeb["Nginx Web Server"]
        MONWeb["Monitoring Client"]
  end
 subgraph AppServers["Application Servers"]
        FWApp["Firewall"]
        SSLApp["SSL Certificate"]
        App["Application Server"]
        Files["Application Files"]
        MONApp["Monitoring Client"]
  end
 subgraph DBServers["Database Servers"]
        FWDB1["Firewall"]
        DBPrimary["MySQL Primary"]
        FWDB2["Firewall"]
        DBReplica1["MySQL Replica"]
        FWDB3["Firewall"]
        DBReplica2["MySQL Replica"]
        MONDB1["Monitoring Client"]
        MONDB2["Monitoring Client"]
        MONDB3["Monitoring Client"]
  end
    User["User (Browser)"] -- "Request www.foobar.com" --> DNS["DNS"]
    DNS -- Returns Cluster IPs --> HAProxyCluster
    FWWeb --> SSLWeb
    SSLWeb --> NginxWeb
    NginxWeb --> MONWeb & FWApp
    FWApp --> SSLApp & App
    SSLApp --> App
    App --> Files & MONApp & FWDB1 & FWDB2 & FWDB3
    Files --> App
    FWDB1 --> DBPrimary & DBPrimary
    FWDB2 --> DBReplica1 & DBReplica1
    FWDB3 --> DBReplica2 & DBReplica2
    DBPrimary --> DBReplica1 & DBReplica2 & MONDB1
    DBReplica1 --> MONDB2
    DBReplica2 --> MONDB3
    HAProxyCluster --> FWWeb
    App -- Response --> NginxWeb
    NginxWeb -- HTTPS Response --> User
    style FWWeb fill:#FFECB3
    style SSLWeb fill:#B3E5FC
    style NginxWeb fill:#C8E6C9
    style MONWeb fill:#D1C4E9
    style FWApp fill:#FFECB3
    style SSLApp fill:#B3E5FC
    style App fill:#FFF9C4
    style Files fill:#BBDEFB
    style MONApp fill:#D1C4E9
    style FWDB1 fill:#FFECB3
    style DBPrimary fill:#FFCDD2
    style FWDB2 fill:#FFECB3
    style DBReplica1 fill:#FFCDD2
    style FWDB3 fill:#FFECB3
    style DBReplica2 fill:#FFCDD2
    style MONDB1 fill:#D1C4E9
    style MONDB2 fill:#D1C4E9
    style MONDB3 fill:#D1C4E9
    style DNS fill:#FFE0B2
    style HAProxyCluster fill:#A5D6A7,stroke-width:2px
    style WebServers fill:#E0F7FA,stroke-width:2px
    style AppServers fill:#E0F7FA,stroke-width:2px
    style DBServers fill:#E0F7FA,stroke-width:2px

```

## How it works

In Task 3, we take the secured and monitored infrastructure from Task 2 and **scale up** by splitting components across different servers and adding a second load balancer for high availability.

1. The user opens their browser and types **<www.foobar.com>**.  
2. The DNS resolves the domain to the **Load Balancer Cluster IPs**.  
3. The request goes to one of the **HAProxy Load Balancers**, which distribute traffic to the appropriate servers.  
4. Servers are now **split by role**:
   - **Web Server** server(s) – handle HTTP/HTTPS requests.  
   - **Application Server** server(s) – run the website logic and application code.  
   - **Database Server** server(s) – host the Primary and Replica databases.  
5. Security (firewalls), SSL, and monitoring clients remain in place for each server.

---

## What’s on the servers

- **Web Server(s)**: Only handle incoming requests and forward them to the application server(s).  
- **Application Server(s)**: Execute the website logic, query the database servers, and return results.  
- **Database Server(s)**: Primary-Replica configuration; Primary handles writes, Replicas handle reads.  
- **Firewalls**: Control incoming and outgoing traffic per server.  
- **SSL Certificates**: Encrypt traffic between users and servers.  
- **Monitoring Clients**: Collect metrics on each server and send to the monitoring service.  

---

## Key Concepts

- **Separation of roles**: Improves scalability and performance. Each server focuses on one task.  
- **Load Balancer Cluster**: Multiple HAProxy nodes in Active-Active mode for high availability.  
- **Primary-Replica Database**: Maintains data integrity while allowing read scaling.  
- **Monitoring and Security**: Ensures we can track performance and protect the infrastructure.

---

## Limitations / Issues

- **Complexity**: More servers mean more management overhead.  
- **Database writes**: Still limited to the Primary DB; failover needs to be handled carefully.  
- **Networking**: More traffic between servers; requires proper network configuration.  
- **Monitoring**: Must be configured to collect metrics from multiple servers efficiently.

---

This architecture **improves scalability and availability** compared to Task 2.  
By splitting servers by role and adding a second load balancer, the system can handle higher traffic and avoid some single points of failure.
