---
layout: post
title:  "포트폴리오- 3. 댓글기능"
date:   2019-02-07 17:15:11 +0000
categories: [portfolio]
tags: [portfolio]
---

<style>
img{
  border : 1px solid #ededed;
}
</style>

포트폴리오- 3. 댓글기능
<br>

>대댓글

대댓글의 방식은 여러가지가 있지만, 구현한 대댓글 기능은 여러 인터넷 카페에서 사용되는 방식을 이용했습니다.
<br>
<img src="/images/petst/comment/comment.jpg" width="600" height="550"><br>

대댓글들은 시간 순대로 나열되어 있으며, 대댓글에 대댓글을 달아도 시간 순대로 나옵니다. 또한 원래 (대)댓글 회원의 닉네임을 대댓글 내용 앞에 붙여 어떤 댓글의 대댓글인지 알 수 있게 하였습니다.
<br>

>(대)댓글 작성

대댓글이 아닌 댓글을 작성할 경우 패런트넘버(freeComments_parent)를 자신의 댓글 넘버( freeComments_commentno)로 둡니다.<br>
대댓글을 작성할 때 가장 상위 댓글의 넘버(freeComments_commentno)를 패런트(freeComments_parent)로 저장하고, 자신이 단 댓글의 내용에 내가 단 대댓글의 주인격의 닉네임을 저장합니다. 참고로 넘버는 AutoIncrease입니다.<br>
예를 들어 a가 댓글을 작성하고,넘버를 1이라고 가정합니다.<br>
b가 a의 댓글에 대댓글을 남겼다면 b의 넘버는 2이며 패런트는 a의 넘버인 1입니다.<br>
이러한 상황에서 c가 b의 대댓글에 대댓글을 남겼다면, c의 넘버는 3이고, 패런트는 가장 상위 댓글인 a의 넘버인 1입니다. 또한, c는 b의 댓글에 대댓글을 단 것이기 때문에 c의 댓글 내용에 b의 닉네임이 저장됩니다.
<br>

>대댓글 list

댓글들을 가져올 때는 해당 게시글에 해당하는 댓글만을 가져와야 하며, 댓글들은 가장 상위에 있는 댓글의 시간 순(parent 순)대로 가져와야하며, 해당 댓글과 관련된 대댓글들을 시간 순(commentno 순)대로 가져와야합니다. <br>
또한 댓글의 1페이지에는 가장 오래 된 댓글 10개가, 마지막 페이지에는 가장 최신의 10개 이하의 댓글들이 나옵니다. <br>
만약 11개의 댓글이 있다면 1페이지에는 10개가, 2페이지에는 하나의 댓글이 나와야하며, 이는 CRUD게시판에서 게시글 목록과는 조금 다릅니다.
<br>다음은 mapper의 코드입니다.
```xml
<select id="selectCommentAll" parameterType="java.util.HashMap"
resultType="freeComment">
  select * from freecomments where freeBoard_boardname=#{freeBoard_boardname} and freeBoard_boardno=#{freeBoard_boardno}
  order by freeComments_parent ASC, freeComments_commentno ASC limit #{comment_skip}, #{comment_numb}
</select>
```

>댓글 삭제

가장 상위의 댓글을 삭제할 때, 만약 대댓글이 없다면 삭제를 하지만 대댓글이 있을 경우 내용을 빈칸으로 update합니다. 그리고 뷰단에서 댓글의 내용이 없는 경우 "삭제된 댓글입니다."라고 나오게 하였습니다. <br>
```java
if(freeComments_commentno == freeComments_parent) { //가장 상위 댓글
  if(bDao.groupCount(freeComments_commentno) == 1)//대댓 없는 경우
    bDao.deleteComments(freeComments_commentno);//지우기
  else {
    HashMap<String, Object> params = new HashMap<String, Object>();
    params.put("freeComments_commentno", freeComments_commentno);
    params.put("freeBoard_content", "");
    bDao.updateComments(params);//대댓있는경우 빈칸으로 업데이트
  }
```

대댓글을 삭제할 때, 만약 가장 상위의 댓글의 내용이 없고(이미 삭제됐고), 자신의 댓글 외에 대댓글이 없다면 가장 상위의 댓글과 자신의 댓글 모두 삭제합니다.
```java
  if(bDao.selectOneComments(freeComments_parent).getFreeComments_content() ==null
  && bDao.groupCount(freeComments_parent) == 2) {
    //가장 상위의 댓글의 내용이 없고(이미 삭제됐고)자신의 댓글 외에 대댓글이 없는 경우
    bDao.deleteComments(freeComments_commentno);//자신의 댓글
    bDao.deleteComments(freeComments_parent);//가장 상위의 댓글
  }
```
그 외(가장 상위의 댓글이 안지워지거나 대댓글이 여러개인경우) 자신의 대댓글만 삭제합니다.
```java
 bDao.deleteComments(freeComments_commentno);
```
<br>
전체 코드
```java
public int deleteComments(int freeComments_commentno, int freeComments_parent) {
  if(freeComments_commentno == freeComments_parent) {
    if(bDao.groupCount(freeComments_commentno) == 1)
      bDao.deleteComments(freeComments_commentno);
    else {
      HashMap<String, Object> params = new HashMap<String, Object>();
      params.put("freeComments_commentno", freeComments_commentno);
      params.put("freeBoard_content", "");
      bDao.updateComments(params);
    }
  }else {
    if(bDao.selectOneComments(freeComments_parent).getFreeComments_content() ==null
    && bDao.groupCount(freeComments_parent) == 2) {
      bDao.deleteComments(freeComments_commentno);
      bDao.deleteComments(freeComments_parent);
    }
    else
    bDao.deleteComments(freeComments_commentno);
  }
  return freeComments_parent;
}
```
