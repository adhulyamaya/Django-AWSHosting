# Django AWS Hosting Procedure

**1. Order Intake & Requirements:**

* Confirm Django version, database, dependencies, domain, SSL.
* Assess traffic expectations.
* Discuss security needs.

**2. AWS Setup:**

* Create/access AWS account.
* Create IAM user (EC2, S3, RDS).
* Create key pair (.pem).
* Create security group (SSH, HTTP/HTTPS, Django port).
* Launch EC2 instance (Ubuntu, instance type, key pair, security group).

**3. Application Deployment:**

* **SSH:** `ssh -i key.pem ubuntu@ip`
* **Update:** `sudo apt update && sudo apt upgrade -y`
* **Python/VENV:** `sudo apt install python3 python3-venv python3-pip -y`, `python3 -m venv venv`, `source venv/bin/activate`
* **Transfer:** `git clone repo_url`, `cd project_dir` or `scp -i key.pem -r local remote`
* **PIP:** `pip install -r requirements.txt` (or `pip install django gunicorn ...`)
* **Static:** `python manage.py collectstatic`
* **Migrate:** `python manage.py migrate`

**4. Gunicorn & Nginx:**

* **Gunicorn Service:** `sudo nano /etc/systemd/system/gunicorn.service`
    ```
    [Unit] Description=Gunicorn, After=network.target
    [Service] User=ubuntu, Group=www-data, WorkingDirectory=/path/project, ExecStart=/path/venv/bin/gunicorn --workers 3 --bind 127.0.0.1:8000 wsgi:app
    [Install] WantedBy=multi-user.target
    ```
* **Gunicorn Enable/Start:** `sudo systemctl daemon-reload`, `sudo systemctl enable gunicorn`, `sudo systemctl start gunicorn`, `sudo systemctl status gunicorn`
* **Nginx Install:** `sudo apt install nginx -y`
* **Nginx Config:** `sudo nano /etc/nginx/sites-available/project`
    ```nginx
    server { listen 80; server_name ip_or_domain;
        location /static/ { alias /path/project/static/; }
        location / { proxy_pass [http://127.0.0.1:8000](http://127.0.0.1:8000); proxy_set_header Host $host; proxy_set_header X-Real-IP $remote_addr; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_set_header X-Forwarded-Proto $scheme; }
    }
    ```
* **Nginx Enable:** `sudo ln -s /etc/nginx/sites-available/project /etc/nginx/sites-enabled`, `sudo nginx -t`, `sudo systemctl restart nginx`

**5. Database (If Needed):**

* **Install PostgreSQL:** `sudo apt install postgresql postgresql-contrib libpq-dev -y`
* **Create DB/User:** `sudo -u postgres psql`, `CREATE DATABASE db; CREATE USER user WITH PASSWORD 'pass'; GRANT ALL PRIVILEGES ON DATABASE db TO user; \q`
* **Django Settings:** Update `settings.py` (ENGINE, NAME, USER, PASSWORD).

**6. Security & Optimization:**

* **UFW:** `sudo ufw allow OpenSSH`, `sudo ufw allow 'Nginx HTTP'`, `sudo ufw enable`
* **Certbot (HTTPS):** `sudo apt install certbot python3-certbot-nginx -y`, `sudo certbot --nginx -d domain_or_ip`
* **Static/DB Optimization:** Configure Nginx/database for performance.

**7. Testing & Handover:**

* Test functionality, performance, security.
* Provide documentation.
* Handover credentials (SSH, AWS, DB).
* Configure DNS.

**8. Maintenance & Support:**

* Regular updates: `sudo apt update && sudo apt upgrade -y`
* Monitoring (CloudWatch).
* Backup strategy.
* Ongoing support.

**PEP 668 (If Needed):**

* **Error:** `error: externally-managed-environment`
* **Solution:** `deactivate`, `rm -rf venv`, `python3 -m venv venv`, `source venv/bin/activate`, `pip install --break-system-packages -r requirements.txt` (use with caution) or create a new virtual environment.

**Placeholders:** `key.pem`, `ip`, `repo_url`, `project_dir`, `/path/project`, `/path/venv`, `wsgi:app`, `ip_or_domain`, `db`, `user`, `pass`, `domain_or_ip`.
