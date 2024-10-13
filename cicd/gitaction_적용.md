# github action 해보기

# junglehub 자동 배포 도전

### 사용 이유
- 4일만에 만든 JungleHub에 버그 다수 발견
- 잦은 배포
- 기존 배포 매우 불편
  - ssh 접속해서 git pull 땡겨오고, nohup으로 app.py 실행
- merge하면 자동으로 배포되도록 구성하고 싶어짐

### 계획
- ec2에 배포 sh 만들어두고, 깃액션에서 ssh 접속해서 실행하도록 
- main branch merge시 trigger 발동
- secret 값들 setting에 등록해서 사용

### 트러블 슈팅 

#### ec2에 sh 파일 작성
- 처음에 cd 안해서 애먹음
  - 홈 디렉터리에서 sh를 실행했는데, app.py의 루트 디렉터리가 아니어서 app.py의 모든 상대경로가 깨짐
  - sh에서 cd 명령어를 사용해서 워킹 디렉터리를 변경할 수 있음!
- slack message 까지 추가 완료
### action script
- environment secrets랑 repository secrets랑 다름
    - repository secrets는 그냥 바로 접근 가능한데, environment secrets는 environment: {environment 이름} 이거를 선언해줘야 함
- environment 더 잘 사용해보기   
    [GitHub Actions 워크플로우의 승인 기능 사용하기 :: Outsider's Dev Story](https://blog.outsider.ne.kr/1556)
  
### nohup 표준 출력 
- 실행 잘 되지만, action의 deploy가 끝나지 않고, 계속 flask의 로그 실행
- nohup은 실행한 프로세스의 표준출력을 호스트에게 절달하는데, 이게 로컬에서 실행할 땐 text 파일로 출력되고, ssh로 연결된 상태에서는 표준출력과 연결된 ssh 터미널로 출력이 됨
- 그래서 ssh 세션이 종료되지 않은 것!
- 아래와 같이 /dev/null로 출력을 리다이렉트 시키면 해결. /dev/null은 출력 없음
  - ssh 세션이 끝나지 않아 깃 action의 스크립트가 종료되지 않는 문제
  - nohup에서 output으로 로그를 계속 출력하는데, 이거 때문에 ssh 세션이 종료되지 않음
  - 명령어에 /dev/null로 리다이렉션 해주면 해결. /dev/null는 유닉스 계열에서 데이터를 버리는 역할을 하는 가상 파일임
  ```bash
  nohup ./.venv/bin/python3.9 app.py > /dev/null 2>&1 &
  ```

 ### action script
 - 도커나 aws 기능을 활용하지 않고, sh 파일만 활용하여 간단하게 작성
 ```yaml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    environment: production

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Install SSH client
      run: sudo apt-get install -y openssh-client

    - name: Connect to EC2 and run script
      env:
        SSH_HOST: ${{ secrets.AWS_SSH_HOST }}
        SSH_USER: ${{ secrets.AWS_SSH_USER }}
        SSH_PRIVATE_KEY: ${{ secrets.AWS_SSH_PRIVATE_KEY }}
      run: |
        echo ${SSH_HOST}
        echo "${SSH_PRIVATE_KEY}" > private_key.pem
        chmod 600 private_key.pem
        ssh -o StrictHostKeyChecking=no -i private_key.pem $SSH_USER@$SSH_HOST 'bash ~/run_junglehub.sh'
    - name: Clean up
      run: rm -f private_key.pem
 ``` 
      