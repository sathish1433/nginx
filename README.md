# Nginx Configuration for Load Balancing and SSL

## 🔧 Configuration Code

### /etc/nginx/site-enable/default (ubuntu) 

In Rhel  /etc/nginx/nginx.conf directly add in conf file.

``` nginx
upstream backend_app {      
    ip_hash;            
    server 127.0.0.1:3001;
    server 127.0.0.1:8000;
}

server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;

    location / {
        proxy_pass http://backend_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# Optional: Redirect HTTP to HTTPS
server {
    listen 80;
    server_name localhost;
    return 301 https://$host$request_uri;
}
```

------------------------------------------------------------------------

## 📘 Summary Table

  -----------------------------------------------------------------------
  Section                             Purpose
  ----------------------------------- -----------------------------------
  **upstream backend_app**            Defines two backend servers (3001 &
                                      8000) for load balancing

  **server { listen 443 ssl; }**      Handles HTTPS requests, proxies
                                      them to backend servers

  **ssl_certificate &                 Enable SSL encryption
  ssl_certificate_key**               

  **proxy_pass backend_app**          Sends traffic to the upstream
                                      backend group

  **proxy_set_header**                Preserves original client headers
                                      (for logging or app logic)

  **server { listen 80; }**           Redirects HTTP → HTTPS
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 🧩 1. Upstream Block

Defines a load balancing group named `backend_app`.

-   **upstream backend_app { ... }**
    -   Creates a backend group Nginx can send traffic to.
-   **ip_hash;**
    -   Ensures requests from the same client IP always go to the same
        backend (sticky sessions).

### ⚙️ Algorithm Comparison

  -----------------------------------------------------------------------
  Algorithm                             Behavior
  ------------------------------------- ---------------------------------
  **round-robin**                       Default --- rotates through all
                                        backends equally

  **least_conn**                        Sends traffic to the backend with
                                        the fewest active connections

  **ip_hash**                           Routes requests consistently
                                        based on client IP
  -----------------------------------------------------------------------

-   **server 127.0.0.1:3001;** → First backend server (port 3001)
-   **server 127.0.0.1:8000;** → Second backend server (port 8000)

🧠 **Result:** Nginx forwards requests to either port **3001** or
**8000** depending on the client's IP.

------------------------------------------------------------------------

## 🔒 2. HTTPS Server Block

Handles **HTTPS** traffic (port 443) and forwards it to backend servers.

-   **listen 443 ssl;** --- Enables SSL/TLS on port 443\
-   **server_name localhost;** --- The domain name this server responds
    to\
-   **ssl_certificate** / **ssl_certificate_key** --- Paths to SSL
    certificate and private key\
-   **location / { ... }** --- Proxies all traffic to the upstream
    backend group\
-   **proxy_set_header Host \$host;** --- Passes the original host
    header\
-   **proxy_set_header X-Real-IP \$remote_addr;** --- Forwards the
    client's real IP address

------------------------------------------------------------------------

## 🌐 3. HTTP → HTTPS Redirect

Redirects all HTTP requests to HTTPS.

-   **listen 80;** --- Listens for HTTP traffic\
-   **server_name localhost;** --- Matches HTTP requests\
-   **return 301 https://$host$request_uri;** --- Redirects to HTTPS
    version

🧾 **Example:**

    http://localhost/test → https://localhost/test

------------------------------------------------------------------------

## ✅ Summary

This Nginx setup provides: - Secure **SSL (HTTPS)** access\
- Intelligent **load balancing** across backend servers\
- **Sticky sessions** using `ip_hash`\
- Automatic **HTTP → HTTPS** redirection
