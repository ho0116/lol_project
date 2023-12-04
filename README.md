# League Of Lengend 승패 예측
<div><img src ="https://github.com/ho0116/lol_project/assets/85285367/c2a6ea22-4b9a-4425-8f52-073b15bbdb97"></div>
더매치랩(The Match LAB)에서 가공한 LOL 게임 데이터를 이용하여 승패 예측 프로젝트


## 1. 개 요
League Of Lengend(이하 LOL)는 전 세계에서 즐기는 대표적인 컴퓨터 게임이다. LOL은 e-sports 산업에서 대표적인 콘텐츠로 자리매김하고 있으며, 2022년 항저우 아시안 게임에서는 정식 종목으로 채택될 만큼 그 영향력은 이미 검증되어 있는 사실이다. <a href="https://biz.chosun.com/site/data/html_dir/2020/12/17/2020121702229.html">[1]</a>

LOL을 개발한 Riot Games는 비단 게임 자체의 재미뿐만 아니라 분석할 수 있는 데이터를 무료로 공개하고 있다. Riot Games API에서는 LOL 소환사의 개인적인 게임 정보와 더불어 경기 데이터까지 제공한다. <a href="https://developer.riotgames.com/">[2]</a> 통계적으로 하루 24시간 동안 수집되는 경기 수는 약 20만 건에 달하며, 소정의 절차를 거치면 손쉽게 얻을 수 있다.

이번 프로젝트에서는 20만 건의 하루치 LOL 게임 데이터를 분석하여 승패를 예측하는 모델을 만들어 보고자 한다.

## 2. 데이터
### 2.1 데이터 소스
이번 프로젝트에 활용할 데이터는 온라인 게임 코칭 전문기업인 더매치랩(The Match LAB)에서 가공한 LOL 게임 데이터를 바탕으로 한다. 데이터는 2023년 8월 25일, 9월 15일, 9월 17일 각각 하루 동안 수집된 LOL 경기에 대한 세부 항목들로 구성되어 있다.

### 2.2 탐색적 데이터 분석
데이터는 5대5 솔로 랭크 경기 약 20만 건으로 구성되어 있으며, 포지션별로 데이터가 구분되어 있다. 5가지 포지션에 대한 내용은 다음과 같다.
<div align="left"><img src="https://github.com/ho0116/lol_project/assets/85285367/33e27184-b16f-450c-99e0-cb560e2b6808" width="500"> </div>

| 포지션 |위치|역할|
|--------|---|---|
| 탑(TOP) |상단라인|1대1 전투, 이니시에이팅, 스플릿 푸시|
| 정글(JUG) |중립지역|정글링, 갱킹|
| 미드(MID) |중단라인|로밍, AP딜러|
| 원딜(ADC) |하단라인|AD딜러|
| 서폿(SPT) |하단라인|원딜케어, 시야장악|

역할은 챔피언에 따라 달라 질 수 있다.  <br>

제공된 데이터의 항목은 총 185개로 구성되어 있으며, 전반적으로 다음과 같은 내용으로 정리해볼 수 있다.

| 항목           | 데이터 속성 |
|--------------|--------|
| 플레이어에 대한 데이터 | tier, kills, deaths, assists, cs... |
| 팀에 대한 데이터    | baron, dragon, goldearned...    |
|라인전 전후에 대한 데이터 |cs_at14, cs_af14, kill_at14...|

여기서 중요하게 고려되는 항목들은 다음과 같다. 

| 데이터 속성 | 데이터의 의미| 중요한 이유 |
|:-------|:-------------|:--------|
|duration|게임진행시간(초)|게임 흐름과 전략의 효과를 이해|
|tier|티어|플레이어와 팀의 실력을 대략 측정|
|kda|Kill/Death/Assist의 비율|전투 참여와 생존 능력을 나타내는 중요 지표|
|goldearned|총 획득 골드|챔피언의 성장과 전력을 반영하는 골드 수입 지표|
|turretkills|포탑 킬 수(막타)|포탑 파괴는 팀 골드 획득과 맵 통제력 확보에 중요|
|object|팀 오브젝트 처치 수(바론+드래곤+전령)|게임 승패에 큰 영향을 미치는 이점을 제공|
|cs|CS 수 (jungle+minion)|챔피언 성장에 중요한 골드와 경험치를 제공|
|dealt|챔피언에게 넣은 데미지|플레이어의 활동량과 전투 참여도를 표현|
|visionscore|시야 점수|적 위치 파악과 전략 결정에 중요한 맵 정보를 제공|
|diffgold|gold 격차|게임 흐름, 승패 판단, 팀의 성장과 전력 차이를 표현|

### 2.3 데이터 전처리
20만 건의 경기 데이터는 수준 별로 편차가 어느정도 존재할 것으로 예상된다. 따라서 일정 수준 이상의 데이터로 지표의 상관성을 통해 승패예측 모델을 만드는 것이 합리적일 것이다.

이에 이번 프로젝트에서는 다음과 같이 정의되는 LOL게임의 등급체계(tier)를 바탕으로, 모든 플레이어가 플래티넘 이상인 경기를 추출하여 분석해보고자 한다.

* 티어에 대한 정보 (표)
* 티어별 히스토그램
* 플래티넘 이상인 경기의 수

도출된 플래티넘 이상의 경기에 대해서, 핵심 데이터 속성으로 ???, ???, ??? 등을 추출했다. 그 이유는 ~~~때문이다. 그리고 승패를 예측하는 것이기 때문에, 경기 데이터를 기준으로 데이터 속성별 정규화(normalize)를 수행했다. 

### 2.4 데이터 프레임 설계

탐색적 데이터 분석과 데이터 전처리를 통해 다음과 같은 데이터 프레임을 만들고자 한다.

| Id  | 팀  | 주요지표 | TOP |MID|
|-----|-----|---------|-----|---|
| 고유번호 | Red/Blue | kda, dpd, ...| ... |...|

## 3. 승패예측 모델

### 3.1 모델 개요
CNN을 썼다. 단순한 합산을 썼다.

### 3.2 성능
플래티넘 이상 4.5만건
데이터 전체 20만건

### 3.3 소결
* 성능에 대한 의미
* 핵심 데이터 항목에 대한 추정 및 분석

## 4. 결론 및 배운점
