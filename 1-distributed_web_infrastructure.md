# Distributed Web Infrastructure – Explanation

## How the request flows

```mermaid
---
config:
  layout: dagre
---
flowchart BT
 subgraph Server1["Server 1"]
        Nginx1["Nginx (Web Server / Reverse Proxy)"]
        App1["Application Server"]
        Files1["Application Files (Code Base)"]
        DB1["MySQL Primary"]
  end
 subgraph Server2["Server 2"]
        Nginx2["Nginx (Web Server / Reverse Proxy)"]
        App2["Application Server"]
        Files2["Application Files (Code Base)"]
        DB2["MySQL Replica"]
  end
 subgraph Servers["Application Servers Cluster"]
        Server1
        Server2
  end
    User["User (Browser)"] -- "Request www\.foobar\.com" --> DNS["DNS"]
    DNS -- Returns LB IP --> HAProxy["HAProxy Load Balancer"]
    Nginx1 --> App1
    App1 --> Files1 & DB1 & Nginx1
    Files1 --> App1
    DB1 --> App1 & DB2
    Nginx2 --> App2
    App2 --> Files2 & DB2 & Nginx2
    Files2 --> App2
    DB2 --> App2
    HAProxy --> Nginx1 & Nginx2
    Nginx1 -- HTTPS Response --> User
    Nginx2 -- HTTPS Response --> User
    style Nginx1 fill:#C8E6C9
    style App1 fill:#FFF9C4
    style Files1 fill:#BBDEFB
    style DB1 fill:#FFCDD2
    style Nginx2 fill:#C8E6C9
    style App2 fill:#FFF9C4
    style Files2 fill:#BBDEFB
    style DB2 fill:#FFCDD2
    style DNS fill:#FFE0B2
    style HAProxy fill:#A5D6A7
    style Servers fill:#E0F7FA,stroke-width:2px

```

## How it works

Now, we improve our simple web stack by distributing it across multiple servers. The website is still <www.foobar.com>
, but instead of a single server, we now have:

A Load Balancer (HAProxy) that receives all incoming requests.

Two servers, each containing:

Web Server (Nginx)

Application Server

Application Files

Database (MySQL)

Here’s how a request works:

The browser asks the DNS server: “What IP address is <www.foobar.com>?”

The DNS responds with the IP of the Load Balancer.

The browser connects to the Load Balancer, which distributes the request to one of the two servers.

## What’s on the servers

Each of the two servers contains everything needed to process requests independently:

Web Server (Nginx): Receives requests from the Load Balancer, serves static files, and forwards dynamic requests to the application server.

Application Server: Runs the website logic and generates responses.

Application Files: The code base of the website.

Database (MySQL): Stores website data. We use a Primary-Replica setup:

Primary: handles write operations.

Replica: receives updates from the Primary and can serve read requests to reduce load.

## Key Concepts

Load Balancer (HAProxy): Distributes traffic between multiple servers to improve availability and reduce overload.

Distribution Algorithm (Round Robin): Each new request is sent to the next server in order, cycling continuously.

Active-Active vs Active-Passive Load Balancer:

Active-Active: Both load balancers handle traffic simultaneously.

Active-Passive: One load balancer is standby and only takes over if the active one fails.

Primary-Replica Database Cluster:

Primary accepts writes; replica handles reads and keeps synchronized with the primary.

Helps scale database reads and improve fault tolerance.

Application Servers: Run the website independently, reducing the risk that one failure takes down everything.

## Limitations / Issues

Single Point of Failure (SPOF):

The Load Balancer is still a single point of failure unless we cluster it.

Security issues:

No firewall or HTTPS configured. Traffic is unencrypted and servers are exposed.

No monitoring:

Performance metrics and alerts are not collected yet.
