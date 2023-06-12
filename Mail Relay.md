Cấu hình SPF & DKIM trước sau đó mới cấu hình Mail Relay

B1. Cấu hình SPF:

+ Trên trang quản lý DNS tạo 1 record dạng TXT : 

VD:
Type	Name	                Content/Value                           TTL
TXT	@ hoặc tên miền	        v=spf1 ip4:Your-IP-WAN mx ~all	        Auto

B2. Cấu hình DKIM:

Để lấy được khoá DKIM bạn hãy SSH vào Server Mail sau đó chạy lệnh sau.

*** Thay my-domain bằng do-main của mình***

su zimbra
/opt/zimbra/libexec/zmdkimkeyutil -a -d my-domain

Nếu đã có DKIM rồi mà bạn muốn xem lại thì dùng lệnh sau

/opt/zimbra/libexec/zmdkimkeyutil -q -d my-domain

