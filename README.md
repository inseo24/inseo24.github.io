### blog blog yahoo~!

[블로그 구경하기](https://inseo24.github.io)

블로그 만들었다 눈누난나 
근데 이거 어떻게 자동으로 주소 생기는 거지! -> 와 이거 github action 으로 돌아가네 빌드 에러 자꾸 난다 이거 이렇게 push 되자마자 배포되는 거 짱이다...  가 아니고 깃헙 페이지로 돌아가는 거였구나. 이거 깃헙 액션으 설정해봤는데 잘 되는지 테스트를 해보자

[![Jekyll CI](https://github.com/inseo24/inseo24.github.io/actions/workflows/jekyll.yml/badge.svg)](https://github.com/inseo24/inseo24.github.io/actions/workflows/jekyll.yml)

<br />

- Job 셋업하고 -> 
- Jekyll pull 해오고 -> 
- checkout(점검? 에서 기존 컨텐트 삭제하고 다시 레포 초기화해서 레포지터리 fetch 해온다. -> 
- 그 다음에 빌드하는데 docker를 쓰는구나 흠믐 여기 소스 코드를 도커 이미지로 만들어서 올리는 건가? 도커에 깃헙 액션 옵션 엄청 붙어있네
- 소스 코드 렌더링하고 이래저래 과정들이 있구만
- tar 실행 (tar는 리눅스에서 주로 사용하는 압축 형식이라고 한다고 그렇다 한다~)
- artifacts 업로드하고 job cleanup을 하는구나
- 마지막에 이전 프로세스들 clean up하고! 와 신기해!


<br/>

카테고리 문제는 어느 정도 방법으 찾았다!
