# Access Control

# Definition

Access Control and Authorization mean the same thing . 

govern(chi phối ) the method and conditions of the enforcement by which subjects(users, devices, or process)<theo chủ thể nào> are allowed or restricted from connecting with , viewing , consuming , entering into or making use .

# Access Control Method ( )

- **Discretionary Access Control** : `<kiểm soát truy cập tùy ý  >` : DAC

Are based on the identity , they are discretionary because owner of resource can be passing it’s permission to other subject .

- **Mandatory Access Control** : `<Kiểm soát truy cập bắt buộc>`MAC

Opposed to DAC ,  Permission access to resource is determine by decision based on security policy but end-user do -not make decision who access it’s resource 

- **Role-based access controls** `<Kiểm soát truy cập dựa vào vai trò>` RBAC

 Are based on the roles played by users and groups in organizational functions

 Based on role of user  , not on information or policy .

Example : 

Trong một hệ thống quản lý tài liệu , những người có vai trò “  Người biên tập “ có thể quyền chỉnh sửa và cập nhật tài liệu , trong khi vai trò  “ Người đọc “  chỉ có quyền xem tài liệu 

- **Attribute-based access controls `<Kiểm soát truy cập dựa vào thuộc tính >` ABAC**

![Untitled](Access%20Control%205550a524c3d44ea9a73139814bb5c69b/Untitled.png)

# What is the bug  ?

Broken access control vulnerabilities exist when a user can access resources or perform actions that they are not supposed to be able to.

# HOW MANY TYPE ?

- Horizontally (user can access to data of other user with it’s  privileged)
- Vertically (user can access to un-permit function )

## HOW TO TEST ?

- Access resources and conduct operations horizontally.
- Access resources and conduct operations vertically.

## Horizontally

- Register two users with identical privileges (Session Active)
- For every request change parameter and session ⇒ analysis response ( behavior of application)
- application will be vulnerable if response is the same

For Example : 

For example, suppose that the `viewSettings` function is part of every account menu of the application with the same role, and it is possible to access it by requesting the following URL: `https://www.example.com/account/viewSettings`. Then, the following HTTP request is generated when calling the `viewSettings` function:

```
POST /account/viewSettings HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=USER_SESSION

username=example_user
```

Change session of second user ⇒ 

```jsx
POST /account/viewCCpincode HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=ATTACKER_SESSION

username=example_user
```

 

if the same content ⇒ vulnerable 

```jsx
HTTP1.1 200 OK
[other HTTP headers]

{
  "username": "example_user",
  "email": "example@email.com",
  "address": "Example Address"
}
```

`Automatic` ⇒ Auth analyzer 

## VERTICALLY

- Create two user with different privileged ( guest-user, user-admin)
- Discovery path or `url` relate to admin functions

TESTING for access to Administrator Functions 

Suppose : 

`addUser` function is the part of the administrative menu of the application ,

`url`  ⇒ `https://www.example.com/admin/addUser`.

```
POST /admin/addUser HTTP/1.1
Host: www.example.com
[...]

userID=fakeuser&role=3&group=grp001
```

Make some assumptions  :

- Whether if the non-admin access this url ?
- What Defend mechanism in situation ?

?? 

## Testing for special Request Header Handling

TYPE : RESTRICTION BASE ON `REQUEST URL` 

For example :

Blocking access form internet to an administrator console exposed /console or /admin 

How to detect if the web application use this defense mechanism ? 

Using HTTP-header :  `X-Original-Url`  or `X-Rewrite-URL` 

- STEP 1:  Send normal request ⇒ content
- STEP 2: Send request with an X-Original-URl header to Non - Existing Resouse

```
GET /index.php HTTP/1.1 =>đã có content
Host: www.example.com
X-Original-URL: /donotexist1 => 404 => handling url 
[...]
```

IF ⇒ 404 , resouce not found  ⇒  confirm ⇒this indicates that the application supports the special request headers

### Other headers to Consider

- Headers:
    - `X-Forwarded-For`
    - `X-Forward-For`
    - `X-Remote-IP`
    - `X-Originating-IP`
    - `X-Remote-Addr`
    - `X-Client-IP`
    
    Values
    
    - `127.0.0.1` (or anything in the `127.0.0.0/8` or `::1/128` address spaces)
    - `localhost`
    - Any [RFC1918](https://tools.ietf.org/html/rfc1918) address:
        - `10.0.0.0/8`
        - `172.16.0.0/12`
        - `192.168.0.0/16`
    - Link local addresses: `169.254.0.0/16`

# HOW TO PREVENT IT  ?

- **Ensure Lookup IDs are not accessible even when guessed or can be Tampered with**

Ví dụ :  Ngay khi biết user-id của người khác ví dụ như `carlos`  là  8765-4a4c-5af6-8d6f-987886

thì khi truy cập vào [http://example.com/user-profile?user-id=8765-4a4c-5af6-8d6f-987886](http://example.com/user-profile?user-id=8765-4a4c-5af6-8d6f-987886) ⇒  không thể truy cập vào được  

⇒ Mitigations : 

1. avoid exposing identifiers to the user when possible. < instead of using JWT or server-side session>
- Unless a resource is intended to be publicy accessible , deny access by default
- At the code level ,  make it mandatory  for developer to declare the access that is allowed for each resource , and deny access by default

?? 

- ABAC , dưới RBAC ,
- unit test <code function> dựa vào đó thì mình test
- Code-Ql

## HOW TO IMPLEMENT RBAC IN WEB APPLICATION  ?

## LAB PORTSWIGGER

## Note :

Chiều ngang  :  Cùng cấp , cùng chức năng , (có những cái chỉ có người dùng được access ) ⇒ không được access chéo ⇒ <chức năng đó nó vẫn được dùng > 

⇒  break được ⇒  `horizontal access control` 

IDOR:  <horizontal and vertical >

- Kiểm soát web là `RBAC` <top level>, `ABAC` <những thứ liên quan đến cố định với user>

Broken Access Control khác với Business Logic Bug 

- FLOW :  xem có bao nhiêu user, role , access được những function gì ?
- playground với function
- Idor - un-guessing

## HOW TO IMPLEMENT RBAC ?

- Implement `ROLE`, `PERMISSION` , `ROLE_PERMISSION`, `USER_ROLE`

`ABAC`: 

- Based on property of object access to determine allowed resource