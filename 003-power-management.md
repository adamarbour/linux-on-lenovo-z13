# Power Management

## Delayed Lid Switch
```bash
mkdir -p /etc/elogind/logind.conf.d
echo -e "[Login]\nHoldoffTimeoutSec=60s" > /etc/elogind/logind.conf.d/mods.conf
```
