# Simple Web Stack – Explanation

## How it works

Imagine a user wants to visit our website. They open their browser and type **<www.foobar.com>**.  

1. The browser asks the **DNS** server: “Hey, what IP address is `www.foobar.com`?”  
2. The DNS replies with the IP of our server, which is **8.8.8.8**.  
3. The browser now connects to our server at 8.8.8.8 and sends a request.  

---

## What’s on the server

On our single server, we have everything needed to run the website:

- **Web Server (Nginx)**: This is the first thing that receives the user’s request. It serves static files like images or CSS and also acts as a reverse proxy, forwarding dynamic requests to the application server.  
- **Application Server**: This is where the code of the website actually runs. It takes the request, runs the logic, and generates the right content for the user.  
- **Application Files**: These are the code files of our website, like HTML, PHP, or other scripts.  
- **Database (MySQL)**: This stores all the data, like users, posts, or any other information the website needs. The application server queries it to get or save data.  

---

## How the request flows

```mermaid
---
config:
  layout: dagre
---
flowchart TD
    User["User (Browser)"] -- "Request www\.foobar\.com" --> DNS["DNS"]
    DNS -- "Returns IP 8.8.8.8" --> Nginx["Nginx (Web Server / Reverse Proxy)"]
    Nginx --> App["Application Server"]
    App --> Files["Application Files (Code Base)"] & DB["MySQL Database"] & Nginx
    DB --> App
    Files --> App
    Nginx -- HTTPS Response --> User
    App@{ shape: hex}
    Files@{ shape: rounded}
    DB@{ shape: cyl}
    style DNS fill:#FFE0B2
    style Nginx fill:#C8E6C9
    style App fill:#FFF9C4
    style Files fill:#BBDEFB
    style DB fill:#FFCDD2
