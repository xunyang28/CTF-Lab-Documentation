
| Port | Service | Version       | Checklist                                                                                                                                             |
| ---- | ------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| 22   | ssh     | OpenSSH 8.4p1 |                                                                                                                                                       |
| 80   | http    | Apache 2.4.56 | - Done directory brute forcing ( No interested directories found )<br>- Drupal 9 — primary web application (port 80)                                  |
| 81   | http    | Apache 2.4.56 | - Web Application shows Open Web Analytics.<br>- In source code found its Version 1.7.3<br>- Open Web Analytics 1.7.3 — exploitation target (port 81) |

# Password Notes
```bash
drupal::Dr0oPaAlpsSwd
tracking::TracK!n9AdmiN
```

# Extra Information

```bash
## in port 81, web application source code found version
Open Web Analytics 1.7.3
```