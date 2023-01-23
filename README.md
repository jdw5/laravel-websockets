## Laravel Websockets

### Files of Note
- WebsocketTest.php
- bootstrap.js
- broadcasting.php
- Dashboard.vue

### Installation and Running
```console
composer require beyondcode/laravel-websockets
```
```console
sail artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="migrations"
```
```console
sail artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="config"
```
```console
composer require pusher/pusher-php-server ^7.0
```
- Run the websockets once the `.env`, `config/broadcast.php` and `config/websockets.php` have been configured to be host local (127.0.0.1) and port 6001
```console
php artisan websockets:serve
```
- If using Sail, add `- '${LARAVEL_WEBSOCKETS_PORT:-6001}:6001'` under `laravel.tests` and `ports` to ensure the WS port is exposed
- Once backend has been configured, under `resources/js/bootstrap.js` uncomment the following lines for Echo using Vue
```
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER ?? 'mt1',
    wsHost: import.meta.env.VITE_PUSHER_HOST ? import.meta.env.VITE_PUSHER_HOST : `ws-${import.meta.env.VITE_PUSHER_APP_CLUSTER}.pusher.com`,
    wsPort: import.meta.env.VITE_PUSHER_PORT ?? 80,
    wssPort: import.meta.env.VITE_PUSHER_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_PUSHER_SCHEME ?? 'https') === 'https',
    disableStats: true
});
```

### Production on Forge
- Use the laravel-websockets package through a subdomain with Nginx reverse proxy
- Create a subdomain with SSL certificate on the same server, and associate an A record to the same server as the main site
- Edit the NGINX configuration file to forward the requests from port 443 (SSL) to 6001 (WS) under `location` as per [docs](https://beyondco.de/docs/laravel-websockets/basic-usage/ssl)
```console
  location / {
    proxy_pass             http://127.0.0.1:6001;
    proxy_read_timeout     60;
    proxy_connect_timeout  60;
    proxy_redirect         off;

    # Allow the use of websockets
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
```
- Configure a Daemon to run `php artisan websockets:serve`
    - Set the directory to that of the main application 
- In the main site `.env` change the `PUSHER_HOST` to reference the subdomain
- Verify a 101 status under the main site to ensure it is connected
