
| Port  | Service | Version       | Checklist                                                                                   |
| ----- | ------- | ------------- | ------------------------------------------------------------------------------------------- |
| 21    | ftp     | ProFTPD 1.3.5 | - anonymous login failed.<br>- ProFTPD 1.3.5 version exploitable                            |
| 80    | http    |               | - Directory brute force not interesting information<br>- Webpage expose Ubuntu Default page |
| 81    |         |               | - Pop up login page                                                                         |
| 10000 | http    |               | - Webmin login interface<br>- MiniServ 1.991                                                |

# Password Notes
```bash
demouser::x8rqsPHQ6X98A
```

# Extra Information

```bash
# FTP
ProFTPD 1.3.5 version exploitable
MiniServ 1.991 - found priv and rce exploitable
```