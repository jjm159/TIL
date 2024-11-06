- 다른 프로세스로 실행해야 하지만, 빠른 개발을 위해 같은 프로세스에서 빠르게 개발
- APScheduler 사용, Wrapping Class로 라이브러리 접근 로직 분리

## 코드
```python 
from apscheduler.schedulers.background import BackgroundScheduler

class Scheduler:

    def __init__(self, daemon=True):
        self._scheduler = BackgroundScheduler(daemon=daemon)

    def runSecondsIntervalJob(self, job, seconds):
        self._scheduler.add_job(job, 'interval', seconds=seconds)
        self._scheduler.start()

```

```python 

from repository.dayTotalCommitCountRepository import dayTotalCommitCountRepository
from repository.userCommitCountRepository import userCommitCountRepository
from repository.repositoryProfile import profile_repository
from repository.boardBlockListRepository import boardBlockListRepository
from module.githubApi import GithubApi
from module.scheduler import Scheduler
from model.dayTotalCommitCount import DayTotalCommitCount
from model.userCommitCount import UserCommitCount
from model.boardBlockList import BoardBlockList
import random


class CommitCountScheduler:

    def __init__(self):
        self.api = GithubApi()
        self.scheduler = Scheduler()

    def run(self):
        self.scheduler.runSecondsIntervalJob(
            self.job,
            self.getIntervalSeconds()
        )

    def getIntervalSeconds(self):
        oneMinute = 60
        return oneMinute * 10

    def job(self):

        result = profile_repository.read_all_jungler()

        def convert(user):
            return {
                'userId': user._id,
                'loginId': user.gitId, 
                'accessToken': user.githubaccesstoken
            }

        userList = map(convert, result)

        # userId, totalCommitCount
        resultList = self.api.getAllTotalCommitCount(list=userList)

        # dayTotalCommitCount 업데이트 
        totalCommitCount = sum([ item['totalCommitCount'] for item in resultList])
        currentDayKey = DayTotalCommitCount.makeCurrentDayKey()
        newTotalCommitCount = DayTotalCommitCount(currentDayKey, totalCommitCount)
        dayTotalCommitCountRepository.update(newTotalCommitCount)

        # user count 업데이트
        userCommitCountRepository.updateAllUserCount(resultList)


        # TODO: board block list 로직 추가

        allIndices = []
        for i in range(0,35):
            allIndices.append(i)

        CONSTANT=3

        totalBlockCount = 35
        openRate = (totalCommitCount / CONSTANT) / totalBlockCount 
        if openRate > 1:
            openRate = 1

        openCount = int(totalBlockCount * openRate)
        # blockedCount = totalBlockCount - openCount

        lastBlock = boardBlockListRepository.todayOpenList()

        if lastBlock is not None:
            lastOpenList = lastBlock.openList

            availAbleCount = openCount - len(lastOpenList)
            if availAbleCount < 0:
                availAbleCount = 0

            # 여기서 newAvailableCount만큼 랜덤하게 뽑으면 끝 
            availableList = list(set(allIndices) - set(lastOpenList))

            newOpenList = random.sample(availableList, availAbleCount)

            lastBlock.openList = sorted(lastOpenList + newOpenList)

            boardBlockListRepository.update(lastBlock)

        else: 
            resultList = sorted(random.sample(allIndices, openCount))
            new = BoardBlockList(_id=BoardBlockList.makeKey(), indices=resultList)
            boardBlockListRepository.update(new)


```