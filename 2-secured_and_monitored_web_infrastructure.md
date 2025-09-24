# Secured and Monitored Web Infrastructure – Explanation

## How the request flows

```mermaid
---
config:
  layout: dagre
---
flowchart TD
    User["User (Browser)"] -- "Request www\.foobar\.com" --> DNS["DNS"]
    DNS -- "Returns LB IP" --> HAProxy["HAProxy Load Balancer"]

    subgraph Servers["Application Servers Cluster"]
        subgraph Server1["Server 1"]
            FW1["Firewall"]
            SSL1["SSL Certificate"]
            Nginx1["Nginx (Web Server / Reverse Proxy)"]
            App1["Application Server"]
            Files1["Application Files (Code Base)"]
            DB1["MySQL Primary"]
            MON1["Monitoring Client"]
            
            FW1 --> SSL1
            SSL1 --> Nginx1
            Nginx1 --> App1
            App1 --> Files1 & DB1
            Files1 --> App1
            DB1 --> App1
            App1 --> Nginx1
            MON1 --> App1
        end

        subgraph Server2["Server 2"]
            FW2["Firewall"]
            SSL2["SSL Certificate"]
            Nginx2["Nginx (Web Server / Reverse Proxy)"]
            App2["Application Server"]
            Files2["Application Files (Code Base)"]
            DB2["MySQL Replica"]
            MON2["Monitoring Client"]
            
            FW2 --> SSL2
            SSL2 --> Nginx2
            Nginx2 --> App2
            App2 --> Files2 & DB2
            Files2 --> App2
            DB2 --> App2
            App2 --> Nginx2
            MON2 --> App2
        end

        subgraph Server3["Server 3"]
            FW3["Firewall"]
            SSL3["SSL Certificate"]
            Nginx3["Nginx (Web Server / Reverse Proxy)"]
            App3["Application Server"]
            Files3["Application Files (Code Base)"]
            DB3["MySQL Replica"]
            MON3["Monitoring Client"]
            
            FW3 --> SSL3
            SSL3 --> Nginx3
            Nginx3 --> App3
            App3 --> Files3 & DB3
            Files3 --> App3
            DB3 --> App3
            App3 --> Nginx3
            MON3 --> App3
        end
    end

    DB1 --> DB2
    DB1 --> DB3
    HAProxy --> FW1
    HAProxy --> FW2
    HAProxy --> FW3
    Nginx1 -- "HTTPS Response" --> User
    Nginx2 -- "HTTPS Response" --> User
    Nginx3 -- "HTTPS Response" --> User

    style DNS fill:#FFE0B2
    style HAProxy fill:#A5D6A7
    style FW1 fill:#FFECB3
    style FW2 fill:#FFECB3
    style FW3 fill:#FFECB3
    style SSL1 fill:#B3E5FC
    style SSL2 fill:#B3E5FC
    style SSL3 fill:#B3E5FC
    style Nginx1 fill:#C8E6C9
    style Nginx2 fill:#C8E6C9
    style Nginx3 fill:#C8E6C9
    style App1 fill:#FFF9C4
    style App2 fill:#FFF9C4
    style App3 fill:#FFF9C4
    style Files1 fill:#BBDEFB
    style Files2 fill:#BBDEFB
    style Files3 fill:#BBDEFB
    style DB1 fill:#FFCDD2
    style DB2 fill:#FFCDD2
    style DB3 fill:#FFCDD2
    style MON1 fill:#D1C4E9
    style MON2 fill:#D1C4E9
    style MON3 fill:#D1C4E9
    style Servers fill:#E0F7FA,stroke-width:2px

```

### How it works

In Task 2, we take the distributed web infrastructure from Task 1 and **add security and monitoring**.

1. The user opens their browser and types **<www.foobar.com>**.  
2. The DNS resolves the domain to the **Load Balancer IP**.  
3. The request goes to the **HAProxy Load Balancer**, which forwards it to one of the two servers.  
4. Each server has a **web server, application server, application files, and database**.  
5. We now add **firewalls**, **SSL encryption**, and **monitoring clients** to secure and observe the system.  

---

## What’s on the servers

- **Web Server (Nginx)**: Receives requests and forwards them to the application server. Now, it serves traffic over **HTTPS** using an SSL certificate.  
- **Application Server**: Runs the website logic and communicates securely with the database.  
- **Application Files**: The code base of the website.  
- **Database (MySQL)**: Primary-Replica setup remains the same, with writes going to the primary and reads possibly served by the replica.  

---

## Additional Components and Why

- **Firewalls** (one per server):  
  - Control which traffic can enter and leave the server.  
  - Protect servers from unauthorized access.  
- **SSL Certificate**:  
  - Encrypts all traffic between users and the servers.  
  - Protects sensitive data like passwords or personal information.  
- **Monitoring Clients** (one per server):  
  - Collect metrics such as CPU, memory, disk usage, and request counts.  
  - Send data to a monitoring service (e.g., Sumologic) for analysis and alerting.  

---

## Key Concepts

- **Firewalls**: Allow only authorized traffic, block malicious requests.  
- **HTTPS / SSL**: Encrypts data in transit so attackers cannot sniff it.  
- **Monitoring**: Provides visibility into server performance and application behavior.  
- **Primary-Replica Database**: Primary still handles writes; replica handles reads and keeps data in sync.  
- **Load Balancer**: Continues to distribute traffic across the two servers.  

---

## Limitations / Issues

- **SSL Termination at Load Balancer**:  
  - If SSL is terminated at HAProxy, traffic between HAProxy and servers might be unencrypted.  
- **Single Primary Database**:  
  - Only one server can accept writes; if it fails, writes cannot be processed.  
- **Servers with identical components**:  
  - Each server has web, application, and database. This can be inefficient and harder to scale individually.  
- **Still some security risks**:  
  - Only basic firewalls; other measures like intrusion detection are not implemented.  

---

This architecture **improves security and observability** compared to Task 1.  
Traffic is now encrypted, servers are protected by firewalls, and metrics are collected for monitoring.  

Next, we can **update the diagram** to include the firewalls, SSL, and monitoring clients, while keeping the same cluster and server structure.
