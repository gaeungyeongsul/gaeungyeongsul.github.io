---
layout: post
title:  "2-2. php & mysql로 회원가입하기 : 비밀번호 정규식, 암호화"
date:   2019-03-09 17:15:11 +0000
categories: [php]
tags: [php]
---

회원가입 시 고려해야할 점은 다음과 같습니다.<br>
1. php와 mysql연동
2. 아이디/닉네임 중복 여부
3. 비밀번호 정규식
4. 비밀번호 암호화(bcrypt)
5. 가입 db연결

<br>
1, 2번글은 이전글을 참고하세요!
<br><br>

>3. 비밀번호 정규식

```javascript
var passPattern = /^(?=.*[A-Za-z])(?=.*\d)(?=.*[$@$!%*#?&]){8,}$/;
if(passPattern.test(user_pass)){
  $('#user_password_res').html('사용가능 한 PW입니다.').css("color","black");
}else{
  $('#user_password_res').html('소문자, 숫자, 특수문자를 포함하여 8글자 이상 <br> 입력하세요.').css("color","red");
}
```
- `/ /` :  정규표현식 검증에 사용되는 패턴의 시작과 끝
- `^` :  처음시작하는 부분부터 일치한다는 표시임
- `x(?=y)` : 오직 'y'가 뒤따라오는 'x'에만 대응
  - `(?=.*[A-Za-z])` : `*[A-Za-z]`가 뒤따라오는지 확인
- `*` : 0또는 그 이상의 문자가 연속될 수 있음
  - `*[]` : 앞에 어떠한 글자가 있거나 없거나
- `[]` : []안의 문자 중 하나를 의미
  - `[A-Za-z]` : 대소문자 중 하나
  - `\d` : 숫자
- `{8,}` : 앞의 글자가 8개 이상
- `$` : 패턴의 끝

<br>
<br>

>4. 비밀번호 암호화(bcrypt)

비밀번호를 그냥 DB에 저장할 수 없으니 암호화를 해야합니다.<br>
bcrypt는 애초부터 패스워드 저장을 목적으로 설계되었고, 현재까지 사용되는 가장 강력한 해시 메커니즘 중 하나이다. bcrypt는 보안에 집착하기로 유명한 OpenBSD에서 기본 암호 인증 메커니즘으로 사용되고 있고 미래에 PBKDF2보다 더 경쟁력이 있다고 여겨진다. 라고 [네이버O2](https://d2.naver.com/helloworld/318732)에 잘 나와있습니다.
<br>
php에도 bcrypt 라이브러리를 사용할 수 있습니다. password_hash()는 강력한 단방향 해싱 알고리즘을 사용하여 새 암호 해시를 만듭니다.
```php
$options = [
    'cost' => 12,
];
echo password_hash("rasmuslerdorf", PASSWORD_BCRYPT, $options);

//$2y$12$QjSH496pcT5CEbzjD/vtVeH03tfHKFy36d4J0Ltp3lRtee9HDxY3K
```
다음과 같이 기존의 문자열을 암호화 된 문자열로 변환합니다. 같은 문자열이어도 암호화된 문자열은 다 다르며 암호화된 메시지로는 원본 메시지를 구할 수 없습니다.<br>
cost(가중치)의 값을 높이면 높일수록 더욱더 강력하게 암호화하지만 암호화 하는 시간과 검증 시간이 늘어나므로 상황에 맞게 조절해야합니다. 기본값은 10입니다.

<br>
<br>
이를 사용해 저의 경우 암호화 된 비밀번호를 `$pass`에 저장한 후 DB에 넣었습니다.

```php
$pass = password_hash($user_password, PASSWORD_BCRYPT, ['cost'=>12]);
로 했습니다.
```
<p style="font-size : 12px">참고 : 복호화는 다음글인 로그인 글에서 설명하겠습니다.</p>
<br>

>5. 가입 db연결

위와 같은 처리를 한 후 DB에 값을 넣을 수 있습니다.
<br>

joinForm.php
```html
<form action="join.php" method="post" onsubmit="return joincheck()">
  <p>회원가입</p>
  <table class="join_table">
    <!--생략-->
  </table>
    <input type="submit" value="가입하기">
</form>
```

join.js
```javascript
var joincheck = function(){
	if($('#user_id_res').html() == '사용가능한 아이디입니다.' &&
		 $('#user_nickname_res').html() == '사용가능한 닉네임입니다.' &&
		 $('#user_password_res').html() == '사용가능 한 PW입니다.' &&
		 $('#user_password').val() == $('#user_password2').val() &&
		 $('input:radio[name=user_gender]').is(':checked')
	 ){
		 return true;
	 }else{
		 alert('가입조건을 확인해주세요.');
		 return false;
	 }
}
```
`onsubmit="return joincheck()"`은 submit을 할 때 joincheck()가 실행되고, joincheck()에서의 반환값이 true일 경우에만 action(join.php로 이동)이 실행됩니다.
<br>
가입조건을 만족하여 submit이 됐을 경우 join.php로 form태그 속의 값들이 post 방식으로 넘어갑니다.

`$_POST['name명']`을 받아 insert sql문을 만들어줍니다.<br>
mysqli_query로 쿼리문을 전송한 후 값을 받아 옵니다.<br>
만약 실패할 경우 alert창이 뜨도록 하였고<br>
성공할 경우 메인페이지로 이동하도록 하였습니다.<br>
header()는 HTTP response header를 직접 조작할 수 있게 하며<br>
exit를 통해 불필요한 코드의 실행을 방지합니다.

```php
$sql = "insert into user(user_no, user_id, user_password, user_nickname ,
   user_gender, user_level, user_join_date)VALUES(0, '$user_id', '$pass',
   '$user_nickname', $user_gender, 1, now())";

$result = mysqli_query($conn,$sql);
mysqli_close($conn);
if($result===false){
 echo "<script>
      alert('회원가입에 실패하였습니다.');
      </script>";
}else{
   header('Location: ../main/main.php');
   exit;
}
```
참고 : join.php 풀 코드
<br>
join.php
```php
include("../db/db.php");
$conn = dbconn();

$user_id = mysqli_real_escape_string($conn, $_POST['user_id']);
$user_password = mysqli_real_escape_string($conn, $_POST['user_password']);
$user_password2 = mysqli_real_escape_string($conn, $_POST['user_password2']);
$user_nickname = mysqli_real_escape_string($conn, $_POST['user_nickname']);
$user_gender = mysqli_real_escape_string($conn, $_POST['user_gender']);

if($user_password == $user_password2){
  $pass = password_hash($user_password, PASSWORD_BCRYPT, ['cost'=>12]);
    $sql = "insert into user(user_no, user_id, user_password, user_nickname , user_gender, user_level, user_join_date)VALUES(0, '$user_id', '$pass', '$user_nickname', $user_gender, 1, now())";
    $result = mysqli_query($conn,$sql);
    mysqli_close($conn);
    if($result===false){
      echo "<script>
        alert('회원가입에 실패하였습니다.');
        </script>";
    }else{
      header('Location: ../main/main.php');
      exit;
    }
}else{
  echo "<script>
  alert('회원가입에 실패하였습니다.');
  history.back();
  </script>";
}
```
