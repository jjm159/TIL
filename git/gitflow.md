크게 두 가지 흐름으로 나눠 개발한다. 하나는 master. 다른 하나는 develop. master는 수정하지 않는다. 원본을 그대로 가지고 있는다. 그리고 개발은 develop 브랜치에서 한다. 개발을 하면서 작은 기능을 구현할 때 마다 feature branch를 만들어서 이 안에서 개발을 진행한다. 

이렇게 진행하고 이제 개발이 완료된 feature는 develop에 한번 merge하고 이를 다시 master에 병합하는 줄 알았지만 그러면 안된다. 중간에 release branch를 다시 만들어서 여기에 병합을 해주어야 한다. 이 배포 버전을 통해 버그나 추가 기능에 대한 수요가 발생하면, 이 release branch에서 개발을 이어간다. 물론 여기서도 개발은 develop branch를 만들어서 진행한다. 

실제로 배포하게 될 코드가 개발이 완료되면 그제서야 master branch에 병합을 하게 된다. 그리고 이 순간이 1.0.0버전이 탄생하는 순간이다. 이를 기념하고 기억하기 위해 tag라는 기능을 통해 master branch의 이 커밋이 1.0.0 버전이라고 기록을 해놓는다. 

만약 출시 버전에서 버그가 발생한 경우 이를 긴급한 상황으로 보고 hotfix라는 branch를 만들어 급히 수정하고 master branch에 병합한다. 이 때에는 0.0.1과 같이 끝자리 수를 올려주어 tag를 남겨두도록 한다. 긴급하게 해결될 문제가 아니라면 develop branch로 가서 deep한 개발을 이어갈 수도 있다.

---
### 참고
[git을 이용한 프로젝트의 흐름(gitflow) - 지옥에서 온 Git](https://opentutorials.org/module/2676/15606)