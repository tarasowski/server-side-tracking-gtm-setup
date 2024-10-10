# Preview container setup

sudo apt update       
sudo apt upgrade

# nginx
sudo apt install nginx -y
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d preview.tagging.domain.com

sudo vi /etc/nginx/sites-available/tagging.conf
sudo vi /etc/nginx/nginx.conf # change in includes to show to the tagging.conf

server {
    listen 80;
    server_name preview.tagging.mvpfoundry.com;

    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name preview.tagging.domain.com;

    ssl_certificate /etc/letsencrypt/live/preview.tagging.domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/preview.tagging.domain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:8080;  # Forward requests to port 8080
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

sudo ln -s /etc/nginx/sites-available/tagging.conf /etc/nginx/sites-enabled/

sudo nginx -t # check for errors
sudo systemctl restart nginx


sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
exit # need to restart the session


docker run -d \
  --name gtm-preview \
  -p 8080:8080 \
  -e CONTAINER_CONFIG='<CONTAINER ID>' \
  -e RUN_AS_PREVIEW_SERVER=true \
  gcr.io/cloud-tagging-10302018/gtm-cloud-image:stable

docker run -d \
  --name gtm-live \
  -p 8080:8080 \
  -e CONTAINER_CONFIG='<CONTAINER ID>' \
  -e PREVIEW_SERVER_URL='https://preview.tagging.domain.com' \
  gcr.io/cloud-tagging-10302018/gtm-cloud-image:stable


Open a web browser and navigate to http://<your-ec2-public-dns>:8080/healthz. You should see an "ok" response indicating that your server is running correctly.