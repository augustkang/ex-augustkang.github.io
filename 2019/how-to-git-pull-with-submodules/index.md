# How to Git Pull With Submodules


집 데탑에만 블로그 소스가 있는지라 회사 컴에서도 작성할 수 있게끔 내 블로그 repo를 clone하였다.
근데 그냥 clone을 하니 submodule로 등록한 repo는 같이 clone이 안되더라.
구글링을 좀 해본 결과 [링크](http://openmetric.org/til/programming/git-pull-with-submodule/)와 같이
```bash
git submodule update --init --recursive
```
clone 이후에 해당 repo의 root에서 위 명령어를 통해 submodule을 업데이트 해주니 해결이 되었다.

