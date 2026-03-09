
| Port | Service | Version | Checklist                                                                  |
| ---- | ------- | ------- | -------------------------------------------------------------------------- |
| 21   | ftp     |         | + anonymous login success<br>+ no useful information<br>+ unable to upload |
| 80   | http    |         | + Directory Search                                                         |
| 3306 | mysql   |         | + Brute Force username and password                                        |

# Password Notes
```bash
root::whatever
helpdesk::IamHacker123

# found password in tickets for admin panel
helpdesk::helpdesk90621
```

# Extra Information

```bash
# Interesting Directory Listing
/scp/login.php
/login.php
/account.php
/web.config

# Exploitation tries
1. Account Creation Failed 
   "Needed email verification"
2. Service search
```