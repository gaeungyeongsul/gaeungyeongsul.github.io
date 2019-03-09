---
layout: post
title:  "2-3. php & mysql로 로그인하기"
date:   2019-03-09 17:15:11 +0000
categories: [php]
tags: [php]
---

>로그인 하기

<img src="/images/php/user/login.jpg" width="330" height="400">
<br>
<br>
로그인 시 고려해야할 점은 다음과 같습니다.
<br>
1. 비밀번호 복호화
2. 세션

이전 글에서 비밀번호를 암호화 하여 DB에 저장하였습니다.<br>
간략한 로직을 설명하자면<br>
로그인 버튼을 클릭할 때 ajax로 아이디와 비밀번호를 `login.php`로 전송합니다.<br>
login.php에서 DB와 연동하여 해당 아이디의 정보를 가져옵니다.
<br>
그리고 회원이 쓴 비밀번호와 DB의 비밀번호가 같은 지 비교합니다.
<br>
이 때 `password_verify()`를 사용하여 복호화된 비밀번호와 회원이 쓴 비밀번호를 비교합니다.
<br>
비밀번호가 맞다면 session에 아이디값을 넣습니다.
<br>
>1. 비밀번호 복호화

우선 해당 아이디의 정보를 DB에서 가져옵니다.<br>
`$get_password`에 DB에 저장된 암호화된 비밀번호를 저장합니다.

```php
$user_id = mysqli_real_escape_string($conn, $_POST['user_id']);
$user_password = mysqli_real_escape_string($conn, $_POST['user_password']);
$sql = "select user_password from user where user_id='$user_id'";
$result = mysqli_query($conn, $sql);
$row = mysqli_fetch_array($result);
if($row != null){
  $get_password = $row['user_password'];
}
```

<br>
DB에서 가져온 비밀번호와 로그인 화면에서 사용자가 입력한 비밀번호를 비교합니다.
<br>
password_verify()는 입력한 패스워드가 password_hash()로 생성된 해시값에 맞는지 확인해 boolean값을 반환합니다.
```php
if (password_verify($user_password, $get_password)) {
  echo true;
}else{
  echo false;
}
```
<br>

>2. 세션

사용자가 아이디와 비밀번호를 맞게 입력하면 로그인이 되고 다음과 같이 session에 id를 담습니다.<br>
```php
  session_start();
  $_SESSION['user_id'] = $user_id;
```

참고 : 세션을 사용하는 이유
<br>
서버와 클라이언트가 통신을 할 때 마다 서버는 클라이언트가 누구인지 인증을 계속해야 합니다. 만약 세션에 저장된 값이 없을 경우 매 화면마다 계속하여 로그인하여 인증을 해야하기 때문에 문제가 발생합니다.
<br>
세션과 비슷한 쿠키의 경우 사용자의 상태 유지를 위한 정보를 웹 브라우저에 저장하고 웹 서버가 쿠키 정보를 읽어서 사용하므로 속도가 빠르나 보안상의 문제가 발생할 수 있습니다.
<br>
따라서 보안적으로도 우수한 세션에 사용자의 아이디를 저장하여 사용자에 대한 인증을 유지할 수 있기 때문에 세션을 사용합니다.

<br>

>3. 로그인 후 header의 변화

<br>
로그인 전
<br>
<img src="/images/php/user/beforelogin.jpg" width="800" height="100">
<br>
로그인 후
<br>
<img src="/images/php/user/afterlogin.jpg" width="800" height="100">
<br>
<br>
header.php에서 session에 user_id의 여부를 체크하여 값이 있다면 로그아웃/마이페이지가 보이도록 없다면 로그인/ 회원가입 이 보이도록 하였습니다.
<br>
header.php
```html
<?php
  session_start();
  if(empty($_SESSION['user_id'])){
?>
<li><a href="../user/loginForm.php">로그인</a></li>
<li><a href="../user/joinForm.php">회원가입</a></li>
<?php
  }else{
?>
<li><a href="../user/logout.php">로그아웃</a></li>
<li><a href="#">마이페이지</a></li>
<?php
  }
?>
```


<br>
<br>
총 코드<br>
login.js
```javascript
$('#login_btn').click(function(){
  $.ajax({
    method: 'POST',
    url : 'login.php',
    data : {
      user_id : $('#user_id').val(),
      user_password : $('#user_password').val()
    },
    success : function(result){
      if(result){
        location.href='../main/main.php';
      }else{
        alert('잘못된 아이디 혹은 비밀번호입니다.');
      }
    },
    error : function(xhrReq, status, error) {
      alert(error)
    }
  })
})
```
login.php
```php
<?php
  include("../db/db.php");
  $conn = dbconn();
  if (isset($_POST['user_id']) && isset($_POST['user_password'])) {
    $user_id = mysqli_real_escape_string($conn, $_POST['user_id']);
    $user_password = mysqli_real_escape_string($conn, $_POST['user_password']);
    $sql = "select user_password from user where user_id='$user_id'";
    $result = mysqli_query($conn, $sql);
    if ($result) {
      $row = mysqli_fetch_array($result);
      if($row != null){
        $get_password = $row['user_password'];
        if (password_verify($user_password, $get_password)) {
          session_start();
          $_SESSION['user_id'] = $user_id;
          echo true;
        } else {
          echo false;
        }
    }else{
        echo false;
      }
    }else{
      echo false;
    }
  }
?>

```
