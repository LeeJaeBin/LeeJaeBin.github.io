---
layout: post
title: Node js Login Process Passport 사용 안하고 구현하기
---

Passport로 구현해 보기 전에 야매(?)로 구현해 본 로그인 프로세스이다.

```javascript
var express = require('express');
var app = express();
var fs = require('fs');
var bodyparser = require('body-parser');
var session = require('express-session');
var mysql = require('mysql');

var conn = mysql.createConnection({
    host : 'localhost',
    user : 'root',
    password : 'password',
    database : 'dbname'
})

app.use(bodyparser.urlencoded({extended: false}));
app.use(bodyparser.json());

app.use(session({
    secret: '12345678',
    resave: false,
    saveUninitialized: true
}));

app.listen(3000, function(){
    console.log('Server start');
});

app.get('/', function(req, res){
    fs.readFile('html/index.html', function(error, data){
        if(error){
            console.log("Error");
        }
        else{
            res.writeHead(200, {'Content-type' : 'text/html'});
            res.end(data);
        }
    });
});

app.get('/registerPage', function(req, res){
    fs.readFile('html/register.html', function(error, data){
        if(error){
            console.log("Error");
        }
        else{
            res.writeHead(200, {'Content-type' : 'text/html'});
            res.end(data);
        }
    })
})

app.post('/register', function(req, res){
    var id = req.body.id;
    var pw = req.body.pw;
    var sql = "INSERT into `user` (id, pw) VALUES ?";
    var value = [[id, pw]];
    conn.query(sql, [value], function(error, result){
        if(error){
            throw error;
        }
        else{
               console.log("Register success");
            res.redirect('/');
        }
    })
});

app.post('/login', function(req, res){
    var id = req.body.id;
    var pw = req.body.pw;
    var sql = "SELECT pw from `user` WHERE id='"+id+"' and pw='"+pw+"'";

    conn.query(sql, function(error, result){
        if(error){
            throw error;
        }
        if(result.length==1){
            sess = req.session;
            sess.username = id;
            console.log(sess.username);
            fs.readFile('html/success.html', function(error, data){
                if(error){
                    console.log('Server error');
                }
                else{
                    res.writeHead(200, {'Content-type' : 'text/html'});
                    res.end(data);
                }
            });
        }
        else{
            fs.readFile('html/fail.html', function(error, data){
                if(error){
                    console.log('Server error');
                }
                else{
                    res.writeHead(200, {'Content-type' : 'text/html'});
                    res.end(data);
                }
            });
        }
    });
});
```

나도 소스가 더러운건 알고 있다. 하지만 모듈화도 하기 전이고 말했듯이 야매로 만든 것이기 때문에 기능 구현만 시켜놓은 상태이다.

## 기능 설명 및 스크린샷

실행하면 아래와 같이 index 페이지가 뜨게 된다.

<img src="/images/index.PNG">

register 버튼을 누르면 id와 pw를 등록할 수 있는 페이지가 뜬다.

<img src="/images/register.PNG">

다시 index로 돌아와 잘못된 id와 pw를 입력하고 제출을 누를 시,

<img src="/images/fail.PNG">

올바른 id와 pw를 입력했을 시,

<img src="/images/success.PNG">

그리고 login success 화면으로 넘어갈 시에는 아래와 같이 console에 세션 이름을 출력하게 하였다.

<img src="/images/console.PNG">

## 코드 설명

### 사용한 모듈들은 아래와 같다.
* express
* fs
* body-parser
* express-session
* mysql

### 위에서 부터 주요 코드 설명
1. 사용할 모듈 불러오기(1~6줄)
2. mysql database와 연결하기(8~13줄)
3. 세션 설정(18~22줄)
4. index 페이지 출력(28~38줄)
5. register 페이지 출력(40~50줄)
6. register process(52~66줄)
7. login process(68~103줄)<br>
7-1. 입력한 id와 pw가 존재하는지 확인하는 쿼리문 작성<br>
7-2. 결과값이 있을 시 success 출력 후 세션 저장<br>
7-3. 결과값이 없을 시 fail 출력
