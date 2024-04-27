# SQL1 .  Feature Profile

## MODE ENABLE BUG

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled.png)

**Phân tích SRC  :** 

Untrusted data :  `id` 

Unsafe method  :  `db.Client.query` 

Untrusted data chảy trực tiếp vào Unsafe method mà không có bất kì biện pháp bảo vệ nào 

**FLOW :** 

- Ko có ID ⇒  return message : “Missing id parameter”
- Nếu có thì thực hiện truy vấn ⇒ nếu record  = 0  thì  return NOT FOUND , nếu có lỗi thì return error message ⇒  nếu ko thì lấy record đầu tiên

**Exploit :** 

1. **Detect DBMS** 

payload :  `1’ and 1::int=1 —`  

Nếu return về user “Đặng Ngọc Khánh” ⇒  Postgres 

( toán tử :: cast operation chỉ có ở postgresSQL)

1. **Union base** 
    
    Detect được có 9 cột ⇒ 
    

```jsx
-1' order by 9 -- 
```

Match datatype 

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled%201.png)

Xác định được column2 và column3 trả về String 

1. Xác định tên database , user , version 

 

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled%202.png)

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled%203.png)

Database :  social_network 

version :  PostgreSQL 16.2 

user :  postgres 

1. Xác định các table trong database social_network 

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled%204.png)

Table : 

- prisma_migrations
- vulnerable
- user
- user_info
- post
- story
- photo
- video

1. Xác định các column trong table user 

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled%205.png)

6 . Xác định data 

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled%206.png)

## MODE DISABLE BUG

**Phân tích SRC  :** 

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled%207.png)

Ở MODE này dev đã filter một số kí tự nguy hiểm 

Và kí tự mình cần nhất ở đây để escape chính là `‘`  

Câu hỏi : 

- Liệu rằng còn kí tự khác thay thế cho  `‘`  để mình thoát khỏi chuỗi đang giữ `untrusted data`  id của mình không  ?
- Vì nó filter select , Liệu `db.Client.query` có hỗ trợ stack query để mình update password của admin không ?

Sau khi search 1 hồi ⇒ vẫn méo có cách escape được ??  

Nhìn lại src , thấy dòng 32 khi dev trả về message cho client mà không `return`  ở đó , nghĩa là code vẫn chạy và `untrusted data` vẫn được chảy vào `unsafe method`   , nhưng mà ở đây là Blind nên mình sẽ vẫn thử flow xem `pg.client.query` có hỗ trợ stack query không ? 

Và nó có hỗ trợ stack query ⇒ debug tắt bỏ 

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled%208.png)

payload  : `-2’; select pg_sleep(10) —` 

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled%209.png)

Để ý góc màn hình có thời gian phản hồi là 10s 

⇒ Giờ chỉ cần update mật khẩu của cả table user thôi ( cái này phải enum từ bug khác  :)))) 

payload : `-2 ‘ ; update user set password=hashvalue where username=tuyy —` 

![Untitled](SQL1%20Feature%20Profile%20f94cbcbb65904424b1ce3805c98b83b5/Untitled%2010.png)

Câu hỏi chưa giải đáp được ? 

Sẽ update trong thời gian tới , khi bị filter `‘` hoặc `“`  làm sao để thoát khỏi `“${id}”` ?? 

Đoán các keyword khác ?