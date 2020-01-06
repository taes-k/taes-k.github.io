---
layout: post
comments: true
title: clean Git-flow 소개
tags: [git, gitflow]
---

### 들어가기에 앞서  

해당 포스트는 현재 저희팀에서 사용하고있는 Git flow를 소개하고자 합니다.  
저희팀의 git flow는 GitHub flow, GitLab flow의 장점들을 합친 모델을 사용중입니다.   

(참고 1. [GitHub flow, GitLab flow 요약](https://ujuc.github.io/2015/12/16/git-flow-github-flow-gitlab-flow/))
(참고 2. [Git flow](http://woowabros.github.io/experience/2017/10/30/baemin-mobile-git-branch-strategy.html))
    
---

### Git branch network 

![1]({{ site.images | relative_url }}/posts/2020-01-07-clean-git-flow/1.png)    
![2]({{ site.images | relative_url }}/posts/2020-01-07-clean-git-flow/2.png)    

---

### 주요 Branch

- work : 메인 브랜치/ 모든 작업 브랜치는 work branch 합쳐지고, 분기합니다. 해당 브랜치는 삭제될수 없는 프로젝트의 모든 작업 소스를 담고있는 메인 브랜치 입니다.  
- develop :	배포 브랜치/ dev 환경 배포를 위한 branch  
- test : 배포 브랜치/ test 환경 배포를 위한 branch  
- master : 배포 브랜치/ pre, live 환경 배포를 위한 branch  
- feature/ : 작업 브랜치 / 신규 개발을 위한 브랜치  
- bugfix/ :	작업 브랜치 / 오류 수정을 위한 브랜치  
- hotfix/ :	작업 브랜치 / 긴급 수정을 위한 브랜치  
- release/ : 배포 브랜치 / Custom 배포를 위한 브랜치로써, Work 브랜치 위에서 배포가 불가능한 상황일때 사용합니다. (Cherry-pick 을 통한 선택적 배포 진행시)   
  
- 메인 브랜치 (work)는 PR merge만을 허용하는 정책으로, 작업 브랜치에서 작업한 내용들은 반드시 Pull-Request를 통해 메인 브랜치에 합쳐집니다.  
- 배포 브랜치 (develop, test, master)는 언제든 삭제가 가능하고, 원하는 배포 커밋 시점에서 새로 생성을 할 수 있습니다.   
- 작업 브랜치 (feaultre/, bugfix/, hotfix)는 메인 브랜치에 PR된 후에는 제거대상 브랜치가 됩니다.   

---

### Branch 정책

모든 작업 브랜치는 Jira 이슈와 연동이 되어야 합니다.


#### 이슈관리 연동 Process

1\. Jira 이슈 생성  
2\. 분기점 만들기 (Jira 이슈태그 자동생성) : ex) feature/CATALOGADM-1   
3\. 커밋 메시지에 Jira 이슈번호를 붙여 자동 연동 : ex) feat(CATALOGADM-1): 프로젝트 init  
4\.작업 완료 후 work브랜치에 pull-request  
5\. 작업 브랜치 제거   

---

### Rebase 정책

![3]({{ site.images | relative_url }}/posts/2020-01-07-clean-git-flow/1.png)    
![4]({{ site.images | relative_url }}/posts/2020-01-07-clean-git-flow/2.png)   

작업 브랜치 → 메인 브랜치 PR시 반드시 rebase work를 통해 work 최상위로 base를 변경하고, 하나의 커밋으로 통합하여 PR을 진행합니다.   
(git rebase -i : 커밋 통합 기능)   

#### Rebase를 필수로 진행해야하는 이유

- Clean branch network  
- 메인 브랜치에 merge시, 작업자가 직접 conflict를 해결하고 PR을 진행할수 있습니다.   
- 기능당 1-commit을 통해 1 cherry pick → 1 기능이 보장됩니다.  
- 아무리 많은 작업자가 함께 작업을 진행해도, 브랜치 모델이 심플하게 유지되고 새로운 작업자 에게도 러닝커브가 높아지지 않습니다.  
- 그래프가 깔끔하고 단순해져 코드흐름 파악 및 명확한 배포 시점을 정하기 용이합니다.  

---

### Pull Request 정책

- 작업 브랜치 생성  
- 작업 브랜치에서 개발  
- 개발완료후 rebase (git rebase -i) → 해당과정을 통해 충돌 해결 및 하나의 커밋으로 통합  
- 메인 브랜치로 PR  
- 코드 리뷰 (Default reviewer 1명)  
- 작업자가 직접 PR 승인  
- 메인 브랜치에 머지가 완료된 작업브랜치는 제거  
