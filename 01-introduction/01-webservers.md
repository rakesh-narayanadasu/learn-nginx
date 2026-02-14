# Introduction and Objectives

## Introduction

Nginx is often hailed as the Swiss Army knife of web servers for its high performance, scalability, and versatility. Whether you're a web developer deploying static sites or a system administrator orchestrating complex, load-balanced environments, Nginx delivers unmatched speed and reliability.

### Key Use Cases

| Use Case       | Description                                                         |
| -------------- | ------------------------------------------------------------------- |
| Static Hosting | Fast delivery of HTML, CSS, JS, and media files                     |
| Reverse Proxy  | Distribute incoming traffic across multiple backend servers         |
| Load Balancing | Automatically route requests to healthy servers                     |
| API Gateway    | Secure and manage API traffic with rate limiting and authentication |


Before you begin, ensure you have a Linux server (Ubuntu, CentOS, etc.) with sudo or root access and an active internet connection.

# Web Servers

A **web server** combines hardware and software to process client requests and serve web content—HTML, CSS, JavaScript, images, and more back to your browser.

## How a Browser Loads a Web Page

When you type a URL (for example, `google.com`) into your browser:

1. The browser queries the [DNS (Domain Name System)](https://en.wikipedia.org/wiki/Domain_Name_System) to resolve the domain name into an IP address.
2. After obtaining the IP, it establishes a TCP connection to the server.
3. The server receives the HTTP/HTTPS request, gathers the requested assets, and sends a response back.
4. Your browser renders the response, displaying the web page.


> Think of DNS like a phone book—translating human-friendly domain names into machine-friendly IP addresses.


## HTTP vs. HTTPS

Web servers communicate over two main protocols:

* [HTTP (HyperText Transfer Protocol)](https://en.wikipedia.org/wiki/HTTP) – unencrypted
* [HTTPS (HTTP Secure)](https://en.wikipedia.org/wiki/HTTPS) – encrypted with TLS/SSL

Most browsers redirect HTTP requests to HTTPS to protect data in transit.

> Transmitting sensitive information over plain HTTP can expose data to eavesdropping and man-in-the-middle attacks. Always prefer HTTPS.


## Traditional vs. Modern Server Architectures

### Legacy Servers: Apache & IIS

* **Apache HTTP Server** (⟶ mid-1990s) uses a process-per-connection or thread-per-connection model.
* **Microsoft IIS** launched around the same time with a similar architecture for Windows environments.

Spawning new processes for each request leads to increased CPU and memory usage under high load:


### Modern Alternatives

Today's high-traffic sites distribute requests across clusters of servers behind load balancers. Popular event-driven and asynchronous servers include:

| Web Server | First Released | Concurrency Model   | Key Benefit                            |
| ---------- | -------------- | ------------------- | -------------------------------------- |
| Nginx      | 2004           | Event-driven, async | Low memory footprint, high concurrency |
| OpenResty  | 2011           | Nginx + Lua modules | Extensible with Lua scripting          |
| LiteSpeed  | 2003           | Event-driven        | Drop-in Apache replacement option      |
| Caddy      | 2015           | Event-driven, Go    | Automatic HTTPS distribution           |


In this guide, we'll focus on **Nginx**, exploring its architecture, configuration syntax, performance optimizations, and real-world deployment patterns.

## References

* [DNS – Wikipedia](https://en.wikipedia.org/wiki/Domain_Name_System)
* [HTTP – Wikipedia](https://en.wikipedia.org/wiki/HTTP)
* [HTTPS – Wikipedia](https://en.wikipedia.org/wiki/HTTPS)
* [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
* [Nginx Official Documentation](https://nginx.org/en/docs/)
