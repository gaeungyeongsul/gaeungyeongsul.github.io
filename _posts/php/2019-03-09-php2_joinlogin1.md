---
layout: post
title:  "2-1. php & mysql로 회원가입하기"
date:   2019-03-09 17:15:11 +0000
categories: [php]
tags: [php]
---

>회원가입

회원가입 폼
<br>
<img src="/images/php/user/join.jpg" width="400" height="500">
<br>
<br>
예외처리
<br>
<img src="/images/php/user/joinexcept.jpg" width="350" height="300">
<br>
<br>
회원가입 시 고려해야할 점은 다음과 같습니다.<br>
1. php와 mysql연동
2. 아이디/닉네임 중복 여부
3. 비밀번호 정규식
4. 비밀번호 암호화(bcrypt)
5. 가입 db연결

<br>

>1. php와 mysql연동

DB에 저장된 회원 정보를 가져오거나, DB에 값을 저장하기 위해선 우선 mysql과 php를 연결해야합니다.
<br>
DB와 연동하기 위해선 `mysqli_connect`함수를 사용하며 인자로 호스트명, 계정명, DB비밀번호, DB명이 필요합니다.
<br>
연결이 실패하면 메세지를 출력한후 종료합니다. 참고로 만약 `die`함수가 아닌 `exit`함수를 사용했다면 메세지를 출력하지 않고 종료합니다.
<br>
너무 당연한거지만 `return`을 통해 호출한 페이지 종료 후 `$conn`값을 가지고 `dbconn()`함수를 호출한 페이지로 넘어갑니다.
<br>
앞으로 계속 `conn`를 사용하기 때문에 함수로 만들어 계속 호출하는 형식으로 진행했습니다.

db.php
```php
function dbconn()
{
  $host_name="localhost";
  $db_user_id="root";
  $db_name="DB명";
  $db_pw="DB비밀번호";
  $conn = mysqli_connect($host_name,$db_user_id,$db_pw, $db_name);//mysql연결

  if(!$conn){
    die('Connect Error: ' . mysqli_connect_error());
  }
  return $conn;
}
```
<br>

>2. 아이디/닉네임 중복 여부

아이디와 닉네임 중복여부를 확인하기 위해선 DB에 해당 아이디와 닉네임이 있는지 확인해봐야 합니다.
<br>
```javascript
$('#user_id').blur(function(){
  idcheck();
});

var idcheck = function(){
  $.ajax({
    method: 'GET',
    url: 'selectOneUser.php',
    data: {
      user_id : $('#user_id').val()
    },
    success : function(result){
      if(!result){
        $('#user_id_res').html('사용가능한 아이디입니다.').css("color","black");
      }else{
        $('#user_id_res').html('사용불가한 아이디입니다.').css("color","red");
      }
    },
    error : function(xhrReq, status, error) {
      alert(error)
    }
  })
}

```
`selectOneUser.php`에 GET방식으로 `user_id`값을 보냅니다. <br>
`selectOneUser.php`에서 db와 연동하여 db에 이미 같은 아이디가 있다면 true를 없다면 false를 반환합니다.
<br>
selectOneUser.php
```php
include("..경로/db.php");
$conn = dbconn();
if (isset($_GET['user_id'])) {
  $user_id = mysqli_real_escape_string($conn, $_GET['user_id']);
  $sql = "select * from user where user_id='$user_id'";
}
$result = mysqli_query($conn, $sql);
if ($result) {
  $row = mysqli_fetch_array($result);
    if($row == null)
      echo false;
    else
      echo true;
}
```
include('db.php')를 통해 위 db.php에서 만든 dbconn함수를 \$conn 변수에 담습니다.<br>
user_id에 \$_GET['user_id']값을 넣습니다.
<br>
이 때 `mysqli_real_escape_string`은 SQL Injection (사용자가 임의로 sql문 주입하여 데이터베이스를 비정상적으로 조작)를 방지하여 특수 문자를 문자열에서 이스케이프합니다.
<br>
select구문을 만들어 \$sql변수에 담고 데이터 베이스 쿼리를 실행하는 `mysqli_query`함수를 실행합니다. 실행 결과를 \$result에 담습니다.
<br>
그리고 `mysqli_fetch_array`를 사용하여 mysql 서버가 응답한 결과를 배열로 변환합니다. 만약 \$row가 null인 경우 false를 반환하도록 합니다.
<br>
