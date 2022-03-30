---
layout: page
title: commit message에 대한 생각(50/72 rule)
permalink: /etc/commit
position: etc
---


# 커밋 메시지에 대한 고찰(feat. 50/72 rule)

<br/>

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## 시작

<br/>

최근에 [커밋 메시지 주도 개발](https://gyuwon.github.io/blog/2021/04/04/commit-message-driven-development.html)이라는 글을 읽었다. 

트위터에서 어쩌다 들어가게 된 개발자 분의 블로그 글이었는데 좀 관점이 독특하다고 생각했다.

----

<strong>“커밋 메시지 주도 개발” </strong>

두둥!

----

코드를 쓸 때 커밋 메시지를 먼저 작성하고 개발을 한다고.

약간 이상하게 들릴 수 있지만 뒤에 붙은 설명이 내가 현재 내가 하는 방식과 비슷해서 굉장히 설득력 있다고 생각했다.

<br />

일단 나는 글쓴분과 다르게 내가 자신 있게 개발을 할 수 있는 게 거의 없다. 

그래서 코드를 작성하기 전에 어떻게 해야 할지 노트에 글을 엄청 끄적인다.(정리라기 보단 머리속 생각을 뱉어내는 것에 가까운) 그리고 생각이 정리되면 코딩을 한다. 그래서 보통 생각하는 건 몇 일이 걸리고 코딩은 몇 시간 되지도 않는다. (...)

<img width="350" alt="image" src="https://user-images.githubusercontent.com/84627144/160753536-526d1719-41ad-45ba-b237-9c5039b37880.png">
<img width="350" alt="image" src="https://user-images.githubusercontent.com/84627144/160753516-70091337-5d28-40a0-8d30-ea690ece881e.png">

나도 주르륵 코딩할 수 있는 사람이었으면 좋겠다요...

<br />

생각하는 것 자체가 오래 걸리다 보니 커밋 메시지가 사실상 한 방에 끝내는 경우가 있는데 그러다 보니 내가 생각한 내용을 커밋 메시지 하나에 다 담는게 부담스러울 때가 있다. 그래서 커밋을 작게 쪼개는 것에 대해 계속 고민하게 된다. 큰 일을 작게 쪼개는 게 중요한데 나는 아직 그러지 못하고 있는게 항상 좀 아쉽다. 큰 그림에 자꾸 집중하게 된달까. 

여튼 이번 글은 그거에 대한 얘기는 아니고, 그 블로그 글에 [50/72 규칙](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)을 적용한다고 써있었는데 친절하게 링크도 달려 있었다. 영어 글이어서 좀 번역기를 열심히 돌렸는데 블로그에 적어두고 틈틈히 읽어보려고 글을 쓰게 됐다.


<br />

## 50/72 rule을 알아보자!

<br/>

일단 [해당 글을 작성한 사람](https://github.com/tpope)이 누굴까 궁금해서 검색해봤는데 뭔가 엄청난 분인 거 같았다.

팔로워가 16.6k 라니. vim 관련해서 뭔가 플러그인을 만드시는 분 같다.

별이 엄청난게 어마어마한 분인 건 확실하다.

레포지터리를 구경하다가 git-bump 라는 걸 구경했는데 거기 커밋 로그를 보니 또 예시로 참고할만 해서 캡쳐해봤다.

![image](https://user-images.githubusercontent.com/84627144/160754843-a82a8376-8074-4cda-b9b7-724621544db1.png)

<br />

----

일단 50/72 규칙은 이렇다.

- 커밋 메시지를 쓸 때, 첫 줄에는 제목을 50자 이내로 요약해 작성한다.
- 제목은 .을 포함하지 않고 명령형으로 작성한다.
(”Fix bug” , not “Fixed bug” or “Fixes bug”)
- 자세한 내용은 **필요하다면 본문을** 추가하되 72자 이내로 쓴다.
- 본문은 제목에서 한칸 띄우고 쓴다.
- Bullet points 써도 된다.(앞에 -, * 이런 글머리 기호 쓰는 거 괜츈!, 그건 각자 컨벤션 따라)
- indent를 쓸 것

----


vim을 쓰면 자기 vim-git runtime files를 설치해서 textwidth를 아예 설정할 수도 있다고 [깨알 홍보](https://github.com/tpope/vim-git)도 하신다. 

```
:set textwidth=72
```

위의 글자수는 영어를 기반으로 쓰여진 글이라 한글은 완벽히 규칙을 적용하긴 어려울 수도 있다.


<br/>

## commit message template를 적용해보자!

<br/>

그래서 커밋 메시지를 작성하는 에디터 기능을 이용해서 아래처럼 세팅을 할 수 있는 거 같다.

근데 아래는 vscode 기반인데 intellij도 이런 기능 있나?

```json
{
    "[git-commit]": {
        "editor.rulers": [50, 72,],
        "editor.fontFamily": "NanumGothicCoding"
    }
}
```

하고 검색을 해봤는데 [오 그냥 git —global 옵션으로 템플릿 적용할 수 있네!!!](https://blog.naver.com/rinjyu/222126055400)

commit template을 만들어서 git-commit-template.txt에 붙여넣기하고 하면 되는구나.

그래서 호다닥 따라해봤다.

<br/>

<img width="838" alt="image" src="https://user-images.githubusercontent.com/84627144/160759028-4a8b14e9-05ad-4f3d-9fec-33bf0aa56bbd.png">

푸헹헹 여러분도 한 번 링크된 블로그 글 보고 따라해보세요!

오늘도 재미가 있었다!