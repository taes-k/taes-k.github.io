---
layout: post
comments: true
title: github issue를 통해 스크럼 관리하기
tags: [github, issue, scrum]
---

### 들어가기에 앞서

이전에 ['Jira를 통해 스크럼 관리하기'](https://taes-k.github.io/2019/12/07/sw-jira-scrum/) 라는 주제로도 글을 쓴 적이 있습니다. 이후 `github`을 메인으로 프로젝트를 관리하는 팀에 오면서 github으로 스크럼 프로젝트를 관리하는 방법에 대해 정리하고 공유하려고 합니다.

앞서서 말씀드리자면, 애자일한 프로젝트 관리에 특화된 `jira`에서 사용하던것 만큼 `github-issue`를 사용하기에는 어려울 수 있습니다. github 또한 신규 업데이트 및 확장 플러그인을 통해 다양한 기능들을 지원하고 있지만 개인적으로는 스크럼 프로젝트를 관리하기에는 jira의 사용성이 좀 더 좋다고 느껴집니다.

저를 포함한 많은 팀에서 `github-enterprise`에서 사용하고 있을것도 고려하여 이번 글에서는 가능한 github 기본 기능으로 스크럼 프로젝트를 관리하는 방법에 대해 정리해 보도록 하겠습니다.

---

### Sprint

스크럼 프로젝트 관리의 핵심이 되는 `스프린트 보드`를 사용하는 방법에 대해서 먼저 알아보도록 하겠습니다. 팀마다 정해진 주기 (1~8주)마다 이슈를 분리하여 목표를 설정하고 관리 할 수 있도록 하는 보드로서, jira에서는 스프린트 보드 이름 그대로의 기능을 제공하고 있기때문에 사용에 어려움이 없습니다.

![1]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/1.png)   

그렇지만 github에서는 `issues`하나로만 모든 이슈들을 관리하기때문에 스프린트 보드 기능을 제공하지 않습니다. 대신, github에서 그 대안으로 사용할수있는것이 `project` 입니다.

![2]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/2.png)  

jira에서 사용했던것 처럼 마일스톤에 스프린트 보드를 project으로 등록해 이슈들을 해당 스프린트보드에 종속시켜 관리 할 수 있습니다.

![3]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/3.png)  

해야하지만 급하지 않은 이슈들은 프로젝트가 지정되지 않은 상태로 등록해두면, 해당 이슈는 `global backlog`상태로 취급하면 됩니다.
 
---

### Task, Sub-Task

jira에서는 해야하는 `작업`의 단위를 `task` 혹은 `story`라는 네이밍으로 관리를 하게됩니다. 또한 해당 작업의 하위작업을 설정하는 `sub-task`기능또한 존재하여 손쉽게 하위이슈들을 연결해 관리 할 수 있습니다.  

![4]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/4.png)  

github에서는 `issue`들 간에 계층구조가 없어 이슈의 하위이슈를 생성 및 관리 하는데 어려움이 있을수 있습니다. 이때 사용 할 수 있는방법이 `to-do list`를 이용하는 방법입니다. (mark down 문법 `- [ ]` 을 사용하여 to-do list를 생성 할 수 있습니다.)

![5]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/5.png) 

![6]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/6.png) 

하위이슈가 별도의 이슈로써 관리되지는 않지만, 하위이슈들의 해결 진행상황에 따라 상위 이슈의 진척률을 확인할수 있다는 점에서 충분히 대안으로 사용할수 있는 방안이라 생각됩니다.

---

### Task flow

jira 에서 태스크를 관리할때에는 일반적으로 `TODO` -> `InProgress` -> `DONE` (custom 추가 가능) 상태를 통해 작업의 진행 척도를 나타낼 수 있습니다. 

![7]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/7.png) 

github 에서는 이슈상태가 `OPEN`, `CLOSE` 으로만 존재하기 때문에 `진행중`, `리뷰중` 등의 상태를 설정하기가 힘들수 있습니다. 위 Sprint를 사용하면서 설정한 project에서 필요한 작업 상태들을 지정해 custom 하게 작업 진행 척도를 등록하여 사용할 수 있습니다.

![8]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/8.png) 


---

### Story point

개인적으로는 jira의 꽃이라고 생각되는 `Story point` 설정입니다. 작업별 업무 난이도, 작업 예상 시간등에 따라 포인트를 설정해 task의 작업량을 예측하고 스프린트별로 정량적으로 프로젝트 작업량을 유추해볼수있는 기능입니다. story point가 잘 설정된 프로젝트에서는 작업자가 생각하는 작업의 난이도 공개, 구성원들이 맡은 작업량이 잘 분배되었는지 확인, 이전 스프린트 대비 팀 전체의 생산성을 확인하는 등의 주요지표로 활용 될 수 있습니다. 

![9]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/9.png) 


github 에서 기본적으로 제공해주는 기능은 없지만 storypoint를 사용할수있는 [크롬 확장프로그램](https://github.com/banyan/github-story-points) 이 존재합니다.  
이슈에 설정된 `[5pt]`예약어를 이용해 이슈마다 storypoint를 지정해줄수 있습니다. 이렇게 지정된 스토리포인트는 task-flow마다 합산하여 확인이 가능합니다.

![10]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/10.png) 

---

### Kanbanboard

jira에서는 task들이 바로 `Kanbanboard`로 연동되어 손쉽게 사용 할 수 있습니다. 게다가 스프린트별로 칸반을 확인하는 등 별도의 작업없이도 편하게 백로그와 칸반을 혼합하여 사용 할 수 있습니다.

![11]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/11.png) 

위에서 sprint를 project 기능을 활용하면서 자연스럽게 칸반보드로 관리 할 수 있게되었습니다. 만약 필요한 태스크 플로우가 있다면 간편하게 칼럼을 추가하여 사용 할 수 있습니다.

![12]({{ site.images | relative_url }}/posts/2022-01-04-github-scrum/12.png) 
