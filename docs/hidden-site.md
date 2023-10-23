# Creating a hidden site

If you want to host Mutiny, but do not want to have it on a generally available url, you can hide Mutiny behind a hidden
url.

## How to do it

Instead of using the provided `nginx.conf` file, you can use the following configuration. This will make sure that
Mutiny is only available on the `/my-hidden-path/` url. You can change this to whatever you want, just switching
`/my-hidden-path/` to whatever you want. 

Do note that the `location ~* ^/(assets|images|public|favicon.ico|manifest.webmanifest)/` block is necessary to make
sure that the static files are still available. If you do not have this block, the static files will not be available
and Mutiny will not work. This means that people could tell you are running Mutiny, even though they cannot access it
without knowing the hidden url.

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;

        # This block will return a 404 not found for direct access to root
        location = / {
            return 404;
        }

        # These files need to be given directly, cannot be behind hidden path
         location ~* ^/(assets|images|public|favicon.ico|manifest.webmanifest)/ {
            proxy_pass http://web:80;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

         location /my-hidden-path/ {
            # To remove the /my-hidden-path part when passing to the proxy
            rewrite ^/my-hidden-path(/.*)$ $1 break;

            proxy_pass http://web:80;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location ~ ^/_services/proxy(/.*)$ {
            rewrite ^/_services/proxy(/.*)$ $1 break;
            proxy_pass http://proxy:8080;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Upgrade $http_upgrade;   # Necessary for WebSocket upgrade
            proxy_set_header Connection "upgrade";    # Necessary for WebSocket upgrade
            proxy_read_timeout 86400;                 # WebSocket connections can be long-lived
        }

        location ~ ^/_services/vss(/.*)$ {
            rewrite ^/_services/vss(/.*)$ $1 break;
            proxy_pass http://vss:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```
