---
layout: post
comments: true
title: Git-flow 적용하기
tags: [git]
---

## Git-flow

Git flow란 이름에서 알수 있다시피, Git의 흐름을 관리하는 방법론입니다.   
그렇다면 git의 흐름을 관리하게되면 어떤 좋은점이 있는지 알아보도록 하겠습니다.  

## Git-flow 히스토리 다이어그램

깃 플로우를 적용하기위해서는 두가지의 필수 사항만 지켜주시면 쉽게 따라하실수 있습니다.  
1. 모든 작업내용은 하나의 메인 브랜치에서 관리한다.  
2. 메인브랜치에 추가될때는 하나의기능, 하나의커밋  

위의 원칙으로 만들어진 Git 하스토리를 살펴보면 다음과 같이 변화된 깔끔한 네트워크를 확인하실수 있습니다.

![1]({{ site.images | relative_url }}/posts/2019-08-09-git-gitflow/1.png)   
  
![2]({{ site.images | relative_url }}/posts/2019-08-09-git-gitflow/2.png)  

## Branch 전략

- master : 실제 라이브 서버와 파이프라이닝 되어있으며, 해당 브랜치는 test 브랜치에서의 pull-request를 통한 머지만을 허용하도록 한다.
- test : QA 서버와 파이프라이닝 되어있으며, 해당 브랜치는 develop 브랜치에서의 pull-request를 통한 머지만을 허용하도록 한다.
- develop : 개발 서버와 파이프라이닝 되어있으며, 해당 브랜치는 work 브랜치에서의 pull-request를 통한 머지만을 허용하도록 한다.
- work : 실제 개발의 루트를 잡고있는 브랜치로서 실제 개발 플로우들의 루트를 관리한다. 단, 이슈 개발에있어서 모든 내용들은 이슈종류에 따라 새로운 브랜치를 만들어 개발후 머지한다.
- feature/ : 신규 기능 개발이슈가 있을때 사용하는 브랜치
- hotfix/ : 긴급 수정 이슈가 있을때 사용하는 브랜치
- bugfix/ : 오류 수정 이슈가 있을때 사용하는 브랜치
  
## Jira 이슈관리 연동

모든 작업들은 Jira 이슈와 매칭하여 작업을 진행하면 이슈관리에 큰 도움이 됩니다.  
### 이슈관리 연동 Process
- Jira 이슈 생성
- 분기점 만들기 (Jira 이슈태그 자동생성) (feature/ISSUE-1 )
- 커밋 메시지에 Jira 이슈번호를 붙여 자동 연동(feat(ISSUE-1): 프로젝트 init)
- 작업 후 work브랜치에 pull-request
- 이슈 브랜치 제거 
- 이슈는 하루작업량을 주기로 생성을 추천

## Rebase

브랜치의 Base를 변경해주는 기능으로, 변경과정에서 새로운 base 브랜치와의 머지과정이 일어납니다.
리베이스의 추가기능으로 해당 브랜치의 여러 커밋 과정을 하나로 합쳐줄 수 있습니다. 이기능을 통해서 이슈들에 대한 개발과정들을 하나의 커밋으로 정리하여  work 브랜치에 pull-request 해 줌으로써, work 브랜치의 업데이트 과정이 한눈에 파악되고 개발 이슈별로 정리된 커밋들을 cherry-pick을 통해 원하는 이슈사항의 개발 코드들만을 가져올 수 있습니다. 이러한 이유로, 이슈별로 하나의 커밋으로 만들어주는 Rebase를 잘 적용 해 주어야 합니다.

- rebase 미적용된 히스토리
![3]({{ site.images | relative_url }}/posts/2019-08-09-git-gitflow/3.png)   
- rebase 적용된 히스토리
![4]({{ site.images | relative_url }}/posts/2019-08-09-git-gitflow/4.png)   

### Rebase 명령어
```
$ git checkout work
$ git pull origin work
$ git checkout feature/AAAA-1-abc
$ git rebase -i work
$ git push --force-with-lease
```

```
pick 5807385 CATALOGADM-1 프로젝트신규생성
s 61ef470 CATALOGADM-2 OAuth2 api 서버사이드 설정
s 9c0eb7c security config 설정, application.yml oauth2 정보 설정 , sasmple rest api controller 생성

# Rebase bf4fb5e..9c0eb7c onto bf4fb5e (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

rebase는 브랜치의 베이스를 강제로 변경해준것이라 그냥 push 를 하면 오류로인해 push가 되지 않습니다.  
그래서 강제 push를 통해 밀어 넣어줘야하는데, `git push -f`를 사용 할 수도 있지만, 안정상 `git push --force-with-lease`를 사용하는것이 좋습니다.  
- 다른 변경이 없는지 확인후 force-push 할수 있도록 해줌
- 참조문서 (https://blog.developer.atlassian.com/force-with-lease/)
rebase 도중 오류가 나면 git rebase --abort 명령어로 취소가 가능합니다.  
  
## Cherry-pick

![2]({{ site.images | relative_url }}/posts/2019-08-09-git-gitflow/2.png)  

위 네트워크에서 1번에서 브랜치를 생성해 feature3을 개발한다고 할때, 이때 현재 브랜치에서는 feature1, bugfix1, feature2에 대해서는 구현이 안되어 있는 상황일텐데,
현재브랜치에 feature2의 기능만이 필요 할때, 체리픽을 통해 feature1, bugfix1에대한 내용은 남겨두고 feature2의 기능만 가져올수 있습니다.  

### Cherry-pick 명령어

```
$ git checkout feature3
$ git cherry-pick --1f212fsdfdvdsv3f3ffsdf12fsdfsdfs (커밋넘버)
```