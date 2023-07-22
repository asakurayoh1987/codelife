
| Profile    | download (kb/s) | upload (kb/s) | latency (ms) |
| -------------- | ----- | ----- | --- |
| Native         | 0     | 0     | 0   |
| GPRS           | 50    | 20    | 500 |
| 56K Dial-up    | 50    | 30    | 120 |
| Mobile EDGE    | 240   | 200   | 840 |
| 2G Regular     | 250   | 50    | 300 |
| 2G Good        | 450   | 150   | 150 |
| 3G Slow        | 780   | 330   | 200 |
| 3G Regular     | 750   | 250   | 100 |
| 3G Good        | 1500  | 750   | 40  |
| 4G/LTE Regular | 4000  | 3000  | 20  |
| DSL            | 2000  | 1000  | 5   |
| Cable 5Mbps    | 5000  | 1000  | 28  |
| Cable 8Mbps    | 8000  | 2000  | 10  |
| 10Mbps shared  | 10000 | 100   | 34  |
| FIOS           | 20000 | 5000  | 4   |
| Wi-Fi          | 30000 | 15000 | 2   |

## Sources:
- https://github.com/mozilla/gecko-dev/blob/master/devtools/client/shared/components/throttling/profiles.js
- https://github.com/WPO-Foundation/webpagetest/blob/master/www/settings/connectivity.ini.sample
- https://github.com/tylertreat/comcast/blob/master/README.md#network-condition-profiles
- https://github.com/dimanech/network-throttling#net-emulation-linux-kernel-only

## Tools:
- https://github.com/tylertreat/comcast
- https://github.com/addyosmani/network-emulation-conditions
- https://github.com/sitespeedio/throttle
- https://github.com/dimanech/network-throttling

## Learn about Network Throttling Profiles
- https://developers.google.com/web/tools/chrome-devtools/device-mode
- https://developer.mozilla.org/en-US/docs/Tools/Network_Monitor/Throttling
- https://docs.microsoft.com/en-us/microsoft-edge/devtools-guide-chromium/device-mode/#throttle-the-network-and-cpu
- https://github.com/GoogleChrome/lighthouse/blob/master/docs/throttling.md
- https://www.debugbear.com/blog/network-throttling-methods
