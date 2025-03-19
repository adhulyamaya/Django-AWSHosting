**1. Gunicorn Systemd Service File (`gunicorn-demo.service`):**

* **Path:** `/etc/systemd/system/gunicorn-demo.service`
* **Content:**

    ```ini
    [Unit]
    Description=Gunicorn application server for demo-project
    After=network.target

    [Service]
    User=ubuntu
    Group=www-data
    WorkingDirectory=/home/ubuntu/demo-project
    ExecStart=/home/ubuntu/demo-project/venv/bin/python /home/ubuntu/demo-project/venv/bin/gunicorn --workers 3 --bind 0.0.0.0:8002 demo_app.wsgi:application
    Environment="PATH=/home/ubuntu/demo-project/venv/bin"
    Restart=always
    RestartSec=3
    StandardOutput=append:/home/ubuntu/demo-project/gunicorn.log
    StandardError=append:/home/ubuntu/demo-project/gunicorn-error.log

    [Install]
    WantedBy=multi-user.target
    ```

* **Commands:**

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable gunicorn-demo
    sudo systemctl start gunicorn-demo
    sudo systemctl status gunicorn-demo
    ```

**2. Nginx Configuration File (`demo.example.com`):**

* **Path:** `/etc/nginx/sites-available/demo.example.com`
* **Content:**

    ```nginx
    server {
        server_name demo.example.com;

        location / {
            proxy_pass [http://127.0.0.1:8002](https://www.google.com/search?q=http://127.0.0.1:8002);
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /static/ {
            alias /home/ubuntu/demo-project/static/; # Replace with your actual static files path
        }

        location /media/ {
            alias /home/ubuntu/demo-project/media/; # Replace with your actual media files path
        }

        listen 443 ssl;
        ssl_certificate /etc/letsencrypt/live/[demo.example.com/fullchain.pem](https://www.google.com/search?q=https://demo.example.com/fullchain.pem); # Replace with your actual certificate path
        ssl_certificate_key /etc/letsencrypt/live/[demo.example.com/privkey.pem](https://www.google.com/search?q=https://demo.example.com/privkey.pem); # Replace with your actual key path
        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
    }

    server {
        if ($host = demo.example.com) {
            return 301 https://$host$request_uri;
        }

        listen 80;
        server_name demo.example.com;
        return 404;
    }
    ```

* **Commands:**

    ```bash
    sudo ln -s /etc/nginx/sites-available/demo.example.com /etc/nginx/sites-enabled/
    sudo nginx -t
    sudo systemctl reload nginx
    ```

**Important Notes (Demo Project):**

1.  **Replace Placeholders:**
    * **Nginx:**
        * Replace `/home/ubuntu/demo-project/static/` and `/home/ubuntu/demo-project/media/` with the actual paths to your demo project's static and media files directories.
        * Replace `/etc/letsencrypt/live/demo.example.com/fullchain.pem` and `/etc/letsencrypt/live/demo.example.com/privkey.pem` with the correct paths to your SSL certificates.
2.  **SSL Certificates:**
    * Ensure that you have obtained SSL certificates for `demo.example.com` using Certbot.
3.  **DNS:**
    * Make sure that your DNS records for `demo.example.com` are pointing to your EC2 instance's public IP address.
4.  **Gunicorn:**
    * Verify that Gunicorn is installed in your virtual environment and that the `demo_app.wsgi` file is correctly configured.
5.  **Firewall:**
    * Ensure that your EC2 instance's security group allows traffic on ports 80 and 443.
6.  **Virtual Environment:**
    * Be sure to activate your virtual environment before running the Gunicorn commands.
7.  **File Names:**
    * Double check the names of the config files.
8.  **Reload:**
    * Always reload systemd after making changes to systemd files.
    * Always reload Nginx after making changes to Nginx files.
9.  **Port 8002:**
    * This demo project uses port 8002, so make sure your Gunicorn is running on that port.
10. **WSGI:**
    * Ensure your `demo_app.wsgi` file contains the correct `application` variable.
11. **Project Name:**
    * All paths and filenames are using `demo-project` as the project name. Adjust as needed.
12. **Domain Name:**
    * All references use `demo.example.com` as the domain name. Adjust as needed.
