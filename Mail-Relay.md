# Trước tiên cấu hình SPF & DKIM

# Cấu hình SPF:

B1. Trên trang quản lý DNS domain tạo 1 record TXT như sau:

Ở đây mình dùng Cloudflare:

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/07048470-b38c-4c8d-b901-c21993c3767e)

# Cấu hình DKIM:

B1. Để lấy được khoá DKIM bạn hãy SSH vào Server Mail sau đó chạy lệnh sau.

**Thay my-domain thành domain của mình**

`su zimbra`

`/opt/zimbra/libexec/zmdkimkeyutil -a -d my-domain`

Nếu đã có DKIM rồi mà bạn muốn xem lại thì dùng lệnh sau

`/opt/zimbra/libexec/zmdkimkeyutil -q -d my-domain`

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/3615b7f7-25b9-46a4-8140-b9d92ea45208)


Bây giờ bạn hãy mở DNS domain và cấu hình như sau:

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/c08f5286-bb5f-4e75-9f2d-0600f8b375a5)

**Dùng [mxtoolbox.](https://mxtoolbox.com/) check dns record**

# ** CONFIGURE MAIL RELAY **

**Hiện tại có rất nhiều đơn vị cung cấp dịch vụ Mail Relay miễn phí, ở đây mình dùng của Brevo https://app.brevo.com/**

**B1. Đăng ký & kích hoạt tài khoản Brevo**

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/c36fafc9-ac0e-4105-aeb3-2c91c3f701f7)

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/e5c14bd1-d93e-4ab1-88cd-48a81677f51d)

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/0a935a3a-3c6d-4b55-bf1c-f0949b95be4d)

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/550610dd-ddfd-458c-a56f-0ea5975b73c9)

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/ee599c62-6b7e-4f18-a834-c549d1913787)

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/c351806f-0df0-4900-b00a-3f7008322f4f)

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/0c12918b-9b87-465d-8ef5-0e8520264a57)

**Sau đó tạo record DNS theo thông tin bên dưới mà Brevo cấp:**
![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/6c286f7d-9600-4160-9fdd-842ae801a254)

**Sau khi add record thấy Brevo code & DKIM code hiện tick xanh là thành công**

**B2. Cấu hình Mail Relay trên Server**

**Lấy thông tin cấu hình mail-relay trên brevo như sau:**

Truy cập website brevo (dashboard) -> Transactional -> Settings -> Configuration -> SMTP Relay -> Get your SMTP Key

![image](https://github.com/trungtm97/Zimbra-Mail/assets/134046186/714ca159-6c81-44d4-aab2-831f9a1b7fae)

`su zimbra`

`zmprov mcf zimbraMtaRelayHost server-mail-relay`

Bây giờ bạn khởi động lại dịch vụ mail zimbra bằng lệnh `zmcontrol restart` để áp dụng các thay đổi

[zimbra@mail ~]$ zmcontrol restart

Host mail.azdigi.online

Stopping zmconfigd...Done.

Stopping imapd...Done.

Stopping zimlet webapp...Done.

Stopping zimbraAdmin webapp...Done.

Stopping zimbra webapp...Done.

Stopping service webapp...Done.

Stopping stats...Done.

Stopping mta...Done.

Stopping spell...Done.

Stopping snmp...Done.

Stopping cbpolicyd...Done.

Stopping archiving...Done.

Stopping opendkim...Done.

Stopping amavis...Done.

Stopping antivirus...Done.

Stopping antispam...Done.

Stopping proxy...Done.

Stopping memcached...Done.

Stopping mailbox...Done.

Stopping logger...Done.

Stopping dnscache...Done.

Stopping ldap...Done.

Host mail.azdigi.online

Starting ldap...Done.

Starting zmconfigd...Done.

Starting dnscache...Done.

Starting logger...Done.

Starting mailbox...Done.

Starting memcached...Done.

Starting proxy...Done.

Starting amavis...Done.

Starting antispam...Done.

Starting antivirus...Done.

Starting opendkim...Done.

Starting snmp...Done.

Starting spell...Done.

Starting mta...Done.

Starting stats...Done.

Starting service webapp...Done.

Starting zimbra webapp...Done.

Starting zimbraAdmin webapp...Done.

Starting zimlet webapp...Done.

Starting imapd...Done.

Khi không cần sử dụng nữa, bạn chỉ đơn giản thực hiện:

`zmprov mcf -zimbraMtaRelayHost server-mail-relay`

**B3. Thiết lặp thep user & password**

`su zimbra`

`echo server-mail-relay youraccount@domain.com:your_password > /opt/zimbra/conf/relay_password` (account & password theo thông tin trên brevo)

`postmap /opt/zimbra/conf/relay_password`

**Kiểm tra lại xem thông tin account & password**

`postmap -q server-mail-relay /opt/zimbra/conf/relay_password`

**Cấu hình zimbra sử dụng username và password này**

`zmprov mcf zimbraMtaSmtpSaslPasswordMaps lmdb:/opt/zimbra/conf/relay_password` #giá trị mặc định là noplaintext,noanonymous

`zmprov mcf zimbraMtaSmtpSaslAuthEnable yes` #giá trị mặc định là no

`zmprov mcf zimbraMtaSmtpCnameOverridesServername no` #giá trị mặc định là no

`zmprov mcf zimbraMtaSmtpTlsSecurityLevel may` #giá trị mặc định là may

`zmprov mcf zimbraMtaSmtpSaslSecurityOptions noanonymous` #giá trị mặc định là noplaintext,noanonymous

**Thiết lập relay Host**

`zmprov mcf zimbraMtaRelayHost server-mail-relay`

**Tiến hành reload lại mta**

 `zmmtactl reload`
