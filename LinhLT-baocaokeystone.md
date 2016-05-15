#Keystone - OpenStack
# Mục lục
- [1. Một số khái niệm](#khai_niem)
	- [1.1 Projects](#projects)
	- [1.2 Domain](#domain)
	- [1.3 Users and Groups](#user_groups)
	- [1.4 Roles](#roles)
- [2. Các thành phần trong Keystone](#thanh_phan)
- [3. Các phương pháp xác thực](#xac_thuc)
	- [3.1 Xác thực bằng mật khẩu](#xacthuc_mat_khau)
	- [3.2 Xác thực bằng token](#xacthuc_token)
- [4. Các loại Token](#token)
	- [4.1 UUID:](#uuid)
	- [4.2 PKI:](#pki)
	- [4.3 PKIZ:](#pkiz)
	- [4.4 Fernet:](#fernet)
		- [4.4.1 Key format](#key_format)
		- [4.4.2 Các loại key](#loai_key)
		- [4.4.3 Generate key](#gen_key)
		- [4.4.4 Rotation Key](#rotation_key)
		- [4.4.4 Token format](#token_format)
		- [4.4.5 Generating token](#gen_token)
		- [4.4.6 Verifying token](#ver_token)
	- [5. Các Backend: Là nơi để lưu trữ, xử lý các yêu cầu.](#backend)
- [6. Cách hoạt động của Keystone](#hoat_dong)
- [Tài liệu tham khảo](#tham_khao)

#Identity Service Projects - Keystone
* Là projects core trong OpenStack.
* Dùng để xác thực, cấp quyền, cung cấp danh sách các dịch vụ cho người dùng, và cho các projects khác trong OpenStack

![](http://i.imgur.com/cjGWu4Q.png)

**=> Ta có thể thấy được rằng, mọi projects trong OpenStack đều phải xác thực thông qua keystone.**

<a name="khai_niem"></a>
#1. Một số khái niệm

<a name="projects"></a>
##1.1 Projects
* Mỗi Projects có nguồn tài nguyên khác nhau.
* Một Projects có thể có nhiều users khác nhau.
* Một user có thể thuộc nhiều Projects khác nhau.
* Users có quyền hạn khác nhau đối với mỗi projects

![](http://916c06e9a68d997cd06a-98898f70c8e282fcc0c2dba672540f53.r39.cf1.rackcdn.com/Screen%20Shot%202014-01-08%20at%201.58.09%20PM.png)

* Ví dụ hình trên: 
    * Mỗi user có thể thuộc nhiều projects khác nhau, và có quyền hạn khác nhau.
    * Ví dụ: Users SandraD, có quyền admin ở trong projects Aerospace nhưng trong projects CompSci chỉ có quyền support.

<a name="domain"></a>
##1.2 Domain
* Bao gồm Projects, Group và Users.
* Một User có thể thuộc nhiều domain khác nhau.
* Users có quyền hạn khác nhau đối với mỗi projects, mỗi domain.

![](http://916c06e9a68d997cd06a-98898f70c8e282fcc0c2dba672540f53.r39.cf1.rackcdn.com/Screen%20Shot%202014-01-08%20at%201.04.26%20PM.png)

<a name="user_groups"></a>
##1.3 Users and Groups
* Groups là một nhóm người dùng.
* Có thể được gán trên domain của group hoặc trên project của group đấy.

![](http://916c06e9a68d997cd06a-98898f70c8e282fcc0c2dba672540f53.r39.cf1.rackcdn.com/ss.png)

* Ví dụ:
	* JohnB có vai trò là Sysadmin ở trong group 1, thuộc 2 Projects Biology và Aerospace.
	* LisaD có vai trò là Engineer trong group 2 thuộc Projects Compsci

<a name="roles"></a>
##1.4 Roles
* Chỉ ra vai trò của người dùng trong project hoặc trong domain,...

![](https://open.ibmcloud.com/documentation/_images/UserManagementWithGroups.gif)

<a name="thanh_phan"></a>
#2. Các thành phần trong Keystone
|Thành phần|Chức năng|
|:--------:|:--------:|
|Identity|Nhận dạng, xác thực user, project, role, metadata|
|Token| Là một chuỗi các ký tự, thẩm định yêu cầu của user đã được indentity|
|Catalog| Chứa danh sách các dịch vụ. Endpoint: Điểm truy cập tới các dịch vụ, thường là địa chỉ url.|
|Policy | Là các chính sách, quy định, quy tắc về project, user,...|

<a name="xac_thuc"></a>
#3. Các phương pháp xác thực

<a name="xacthuc_mat_khau"></a>
##3.1 Xác thực bằng mật khẩu
![](http://i.imgur.com/fXzFnnH.png)

<a name="xacthuc_token"></a>
##3.2 Xác thực bằng token
Là một chuỗi các ký tự, đã được mã hóa nhằm bảo đảm an ninh an toàn thông tin.

![](http://i.imgur.com/ZAK7w99.png)

<a name="token"></a>
#4. Các loại Token

<a name="uuid"></a>
##4.1 UUID:
* Độ dài 32 byte, lưu vào database. Không nén.
* Tuy nhiên, cứ mỗi lần xác thực là phải gửi đến Keystone nên Keystone phải xử lý nhiều, làm giảm hiệu năng.

<a name="pki"></a>
##4.2 PKI:
* Mã hóa bằng Private Key, kết hợp Public key để giải mã, lấy thông tin. Token chứa nhiều thông tin như Userid, project id, service catalog,...
* Tuy nhiên, Header của HTTP chỉ giới hạn 8kb, nên sẽ gặp lỗi.
* Xác thực ngay tại user, không cần phải gửi yêu cầu xác thực đến Keystone.

<a name="pkiz"></a>
##4.3 PKIZ:
* Tương tự PKI.
* Khắc phục nhược điểm của PKI, token sẽ được nén lại để có thể truyền qua HTTP.

<a name="fernet"></a>
##4.4 Fernet: 
* Sử dụng mã hóa đối xưng (Sử dụng chung key để mã hóa và giải mã).
* Không lưu token vào database.			
* Không nén.
* Chứa các thông tin như userid, projectid, domainid, methods, expiresat,....
* Không chứa serivce catalog,...
* Nhanh hơn 85% so với UUID và 89% so với PKI.

|Token Types | UUID | PKI | PKIZ | Fernet|
|:----------:|:----:|:---:|:----:|:-----:|
|Size	|32 Byte	|KB Level	|KB Level	|About 255 Byte|
|Support |local authentication	|not support	|stand by	|stand by|	not support|
|Keystone load|Big	|small	|small|	Big|
|Stored in the database	|Yes|	Yes|	Yes	|no|
|Carry information	|no	|user, catalog, etc.|	user, catalog, etc.|	user, etc.|
|Involving encryption	|no	|Asymmetric encryption|	Asymmetric encryption|	Symmetric encryption (AES)|
|Compress	|no	|no	|Yes|	no|
|Supported	|D	|G	|J	|K|

<a name="key_format"></a>
###4.4.1 Key format
```sh
Signing-key ‖ Encryption-key
```
* Signing-key, 128 bits
* Encryption-key, 128 bits

<a name="loai_key"></a>
###4.4.2 Các loại key
* Primary key: Sử dụng cho mã hóa và giải mã token fernet. (Chỉ số khóa cao nhất)
* Secondary key: Giải mã token. (chỉ số khóa nằm giữa primary key và secondary key)
* Staged key: Tương tự Sencondary key. Khác ở chỗ là Stage key sẽ trở thành primary key ở lần xoay khóa tiếp theo. (Chỉ số khóa thấp nhất).

<a name="gen_key"></a>
###4.4.3 Generate key
Dưới đây là đoạn mã đến sinh ra key

```sh
>>> import base64
>>> import os
>>>
>>> b_key = os.urandom(32) # the fernet key in binary
>>> b_key
'2g\x06\xb3O\xe2D\x7f\x86\xc9\xb0\xb8\xd4\x071v\xd8/\x80\x88\xb8\x92M\xd3\xf7\x86\xc0\xaa\x82\xfb\x97\xe9'
>>>
>>> b_key[:16] # signing key is the first 16 bytes of the fernet key
'2g\x06\xb3O\xe2D\x7f\x86\xc9\xb0\xb8\xd4\x071v'
>>>
>>> b_key[16:] # encrypting key is the last 16 bytes of the fernet key
'\xd8/\x80\x88\xb8\x92M\xd3\xf7\x86\xc0\xaa\x82\xfb\x97\xe9'
>>>
>>> key = base64.urlsafe_b64encode(b_key) # base64 encoded fernet key
>>> key
'MmcGs0_iRH-GybC41AcxdtgvgIi4kk3T94bAqoL7l-k='
```

<a name="rotation_key"></a>
###4.4.4 Rotation Key

![](http://www.mattfischer.com/blog/wp-content/uploads/2015/05/fernet-rotation1.png)

* Hiện tại:
	* Primary key là 2.
	* Secondary key là 1.
	* Staged key là 0.
* Quá trình xoay khóa.
	* Khóa Primary key 2 trở thành khóa Secondary key.
	* Khóa Staged key 0 trở thành khóa Primary key.
	* Khóa Secondary key 1 có thể giữ nguyên hoặc bị xóa đi. Vậy khi nào xóa đi, đó là khi mình cấu hình có tối đa bao nhiêu key trong file `/etc/keystone/`. Nếu cấu hình là 3 key thì Secondary key 1 sẽ bị xóa đi.

<a name="token_format"></a>
###4.4.4 Token format
```sh
Version ‖ Timestamp ‖ IV ‖ Ciphertext ‖ HMAC
```
* Version: 8bits, chỉ phiên bản token được sử dụng. Hiện tại thì chỉ có 1 phiên bản token. Bắt đầu bằng `0x80`.
* timestamp: kiểu nguyên, 64 bits. Là khoảng thời gian từ ngày 1/1/1970 đến ngày mà token được sinh ra.
* IV (Initialization Vector): 128bits. Với mỗi token sẽ có một giá trị IV.
* Ciphertext: Có kích thước khác nhau, nhưng là bội số 128bits. Chứa các thông điệp nhập vào.
* HMAC: có độ dài 256bits, chứa các trường sau
```sh
Version ‖ Timestamp ‖ IV ‖ Ciphertext
```
Cuối cùng Fernet Token sử dụng Base64 URL safe để encoded các thành phần trên.

<a name="gen_token"></a>
###4.4.5 Generating token

Given a key and message, generate a fernet token with the following steps, in order:

* Ghi lại thời gian hiện tại ở trường timestamp.
* Chọn một giá trị IV.
* Xây dựng ciphertext:
	* Pad các tin nhắn là bội số của 128bit (16byte).
	* mã hóa các thông điệp sử dụng AES128-CBC, với tùy chọn IV ở trên và sử dụng encryption-key (trong keyformat).
* Tính HMAC bằng cách sử dụng Signing-key
* Ghép tất cả các trường trên lại với nhau.
* Token được tạo ra bằng cách mã hóa base64url các trường trên


<a name="ver_token"></a>
###4.4.6 Verifying token
* Giải mã base64url token.
* Đảm bảo các bye đầu tiên của mã thông bảo là 0x80 (phiên bản token).
* Nếu người dùng quy định time-to-live cho token thì phải đảm bảo timestamp không phải trong quá khứ.

* Recompute HMAC từ các trường, sử dụng signing-key
* Đảm bảo HMAC được recompute lại phù hợp với trường HMAC lưu trong token
* Giải mã ciphertext sử dụng thuật toán AES/128-CBC với chế độ IV và sử dụng Encryption key.
* Thông điệp ban đầu được giải mã

<a name="backend"></a>
##5. Các Backend: Là nơi để lưu trữ, xử lý các yêu cầu.

![](http://i.imgur.com/bwWVFy6.png)

* Memcached: Phân phối và lưu trữ bộ nhớ cached (bộ nhớ tạm) trên RAM. 
* KVS Backend: 
* SQL Backend: Lưu trữ dữ liệu
* PAM backend: Xác thực người dùng, mối quan hệ giữa user và projects.
* LDAP backedn: 

<a name="hoat_dong"></a>
#6. Cách hoạt động của Keystone

![](http://i.imgur.com/uDzPLna.png)

* 1: User gửi thông tin đến Keystone (Username và Password)
* 2: Keystone kiểm tra thông tin. Nếu đúng, nó sẽ gửi về user 1 token.
* 3: User gửi token và yêu cầu đến Nova.
* 4: Nova gửi token đến Keystone để kiểm tra token này có đúng không? có những quyền hạn gì. Keystone sẽ trả lời lại cho Nova.
* 5: Nếu token có quyền, Nova gửi token và yêu cầu image đến Glance.
* 6: Glance gửi token về Keystone để xác thực và kiểm tra xem user này có quyền với file image này không. Keystone sẽ trả lời đến Glance.
* 7: Nova gửi token, và yêu cầu về mạng đến Neutron.
* 8: Neutron gửi token đến Keystone. Keystone sẽ trả lời cho Neutron là user này có được phép hay không.
* 9: Neutron trả lời cho Nova..
* 10: Nova trả lời cho người dùng.

<a name="tham_khao"></a>
#Tài liệu tham khảo
* *Steve Martinelli, Henry Nash & Brad Topol*: Identity, Authentication & Access Management in OpenStack
* http://www.slideshare.net/openstackindia/openstack-keystone-identity-service
* https://github.com/fernet/spec/blob/master/Spec.md
* https://developer.ibm.com/opentech/2015/11/11/deep-dive-keystone-fernet-tokens/
* http://www.openstack.cn/?p=5120
