---
sidebar_position: 1
tags: ['symfony']
---

# Messenger Worker

## Symfony configuration

```bash
MESSENGER_TRANSPORT_DSN=doctrine://default
# MESSENGER_TRANSPORT_DSN=redis://localhost:6379/messages
# MESSENGER_TRANSPORT_DSN=amqp://guest:guest@localhost:5672/%2f/messages
```

```bash
# config/packages/messenger.yaml
framework:
    messenger:
        transports:
            async: "%env(MESSENGER_TRANSPORT_DSN)%"
```

## Systemd configuration (Linux)
Systemd user service configuration files typically live in a ~/.config/systemd/user directory. For example, you can create a new messenger-worker.service file. Or a messenger-worker@.service file if you want more instances running at the same time:

```bash
vim ~/.config/systemd/user/application-messenger-worker.service
```

```bash
[Unit]
Description=Symfony messenger-consume %i

[Service]
ExecStart=php /var/www/application/bin/console messenger:consume async --time-limit=3600 --env=prod
Restart=always
RestartSec=30

[Install]
WantedBy=default.target
```

Now, tell systemd to enable and start one worker:

```bash
systemctl --user start application-messenger-worker.service
systemctl --user status application-messenger-worker.service
systemctl --user enable application-messenger-worker.service
```

If you change your service config file, you need to reload the daemon:

```bash
systemctl --user daemon-reload
```

[Read more](https://symfony.com/doc/current/messenger.html)