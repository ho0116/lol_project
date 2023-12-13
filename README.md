# League Of Lengend 승패 예측
<div><img src ="https://github.com/ho0116/lol_project/assets/85285367/168d93e9-c8b6-4d13-8007-2cf5816ba80a"></div>
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

| 포지션 |역할|
|--------|---|
| 탑(TOP) |상대편 탑 라이너와 1:1로 전투한다. 자신의 라인을 안정적으로 유지하고 상대편을 압박하며, 상황에 따라 다른 라인으로 로밍하여 팀에 도움을 줄 수도 있다.|
| 정글(JUG) |정글몹을 처치하는데 경로 선택과 몬스터 처치 시기를 잘 결정하여 팀에게 이점을 가져다준다. 갱킹을 통해 적 라이너를 습격하거나 도움을 준다.|
| 미드(MID) |자신의 라인을 유지하면서도 상황에 따라 다른 라인으로 로밍하여 팀에 도움을 준다. AP(Ability Power) 딜러로서 스킬을 사용하여 적에게 마법 데미지를 입힌다.|
| 원딜(ADC) |AD(Attack Damage) 딜러로서 적에게 물리적 공격을 준다.|
| 서폿(SPT) |원딜을 케어하고, 시야를 확보하여 팀의 전략적 우위를 도와준다. 상대편의 원딜과 서폿에게 적절한 CC(제어) 기술을 활용하여 전투에서 유리한 상황을 만든다.|

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
|duration|게임진행시간(초)|게임 흐름과 전략의 효과를 이해하는데 중요한 요소이다.|
|tier|티어|플레이어와 팀의 실력을 대략 측정하는 데 도움이 되므로 중요하다.|
|kda|Kill/Death/Assist의 비율|전투 참여와 생존 능력을 나타내는 중요 지표이다.|
|goldearned|총 획득 골드|게임 동안 획득한 골드는 아이템 구매 및 업그레이드에 중요한 역할을 한다.|
|tower 100,200|팀별 타워 제거 수|골드 획득, 맵 컨트롤, 그리고 해당 지역의 오픈으로 게임진행 및 승리에 영향이 미치는 지표이다.|
|object|팀 바론,드래곤,전령 처치 수|게임 승패에 큰 영향을 미치는 이점을 제공한다.|
|cs 100,200|팀별 정글몹,미니언 처치 수|챔피언 성장에 중요한 골드와 경험치를 제공한다.|
|dealt 100,200|팀별 챔피언에게 넣은 데미지| 플레이어의 전투 참여와 팀의 전략적인 기여를 평가하는 지표이다.|
|visionscore 100,200|팀별 시야 점수|적 위치 파악과 전략 결정에 중요한 맵 정보를 제공한다.|
|diffgold|gold 격차|골드 차이가 클수록 더 강한 아이템을 가진 챔피언이 많아져서 승리에 유리하다.|

### 2.3 데이터 전처리
20만 건의 경기 데이터는 수준 별로 편차가 어느정도 존재할 것으로 예상된다. 따라서 일정 수준 이상의 데이터로 지표의 상관성을 통해 승패예측 모델을 만드는 것이 합리적일 것이다.

이에 이번 프로젝트에서는 다음과 같이 정의되는 LOL게임의 등급체계(tier)를 바탕으로, 모든 플레이어가 플래티넘 이상인 경기를 추출하여 분석해보고자 한다.

<details>
<summary>한국 티어 분포표 <a href="https://www.op.gg/statistics/tiers">[op.gg]</a></summary>
<table>
  <tr align="center"><th>티어</th><th>단계</th><th>분포</th><th>합</th></tr>
  <tr align="center"><th>챌린저</th><td>Ⅰ</td><td>0.01%</td><td>0.01%</td></tr>
  <tr align="center"><th>그랜드마스터</th><td>Ⅰ</td><td>0.02%</td><td>0.02%</td></tr>
  <tr align="center"><th>마스터</th><td>Ⅰ</td><td>0.47%</td><td>0.47%</td></tr>
  <tr align="center"><th rowspan="4">다이아</th><td>Ⅰ</td><td>0.39%</td><td rowspan="4">3.45%</td></tr>
  <tr align="center"><td>Ⅱ</td><td>0.55%</td></tr><tr align="center"><td>Ⅲ</td><td>0.69%</td></tr>
  <tr align="center"><td>Ⅳ</td><td>1.82%</td></tr>
  <tr align="center"><th rowspan="4">에메랄드</th><td>Ⅰ</td><td>1.59%</td><td rowspan="4">13.68%</td></tr>
  <tr align="center"><td>Ⅱ</td><td>1.93%</td></tr><tr align="center"><td>Ⅲ</td><td>3.21%</td></tr>
  <tr align="center"><td>Ⅳ</td><td>6.95%</td></tr>
  <tr align="center"><th rowspan="4">플래티넘</th><td>Ⅰ</td><td>2.15%</td><td rowspan="4">16.97%</td></tr>
  <tr align="center"><td>Ⅱ</td><td>3.22%</td></tr><tr align="center"><td>Ⅲ</td><td>4.21%</td></tr>
  <tr align="center"><td>Ⅳ</td><td>7.39%</td></tr>
  <tr align="center"><th rowspan="4">골드</th><td>Ⅰ</td><td>2.68%</td><td rowspan="4">19.26%</td></tr>
  <tr align="center"><td>Ⅱ</td><td>3.92%</td></tr><tr align="center"><td>Ⅲ</td><td>4.85%</td></tr>
  <tr align="center"><td>Ⅳ</td><td>7.81%</td></tr>
  <tr align="center"><th rowspan="4">실버</th><td>Ⅰ</td><td>2.78%</td><td rowspan="4">19.04%</td></tr>
  <tr align="center"><td>Ⅱ</td><td>4.01%</td></tr><tr align="center"><td>Ⅲ</td><td>4.87%</td></tr>
  <tr align="center"><td>Ⅳ</td><td>7.38%</td></tr>
  <tr align="center"><th rowspan="4">브론즈</th><td>Ⅰ</td><td>3.3%</td><td rowspan="4">19.93%</td></tr>
  <tr align="center"><td>Ⅱ</td><td>4.5%</td></tr><tr align="center"><td>Ⅲ</td><td>5.08%</td></tr>
  <tr align="center"><td>Ⅳ</td><td>7.05%</td></tr>
  <tr align="center"><th rowspan="4">아이언</th><td>Ⅰ</td><td>3.11%</td><td rowspan="4">7.21%</td></tr>
  <tr align="center"><td>Ⅱ</td><td>2.54%</td></tr><tr align="center"><td>Ⅲ</td><td>1.08%</td></tr>
  <tr align="center"><td>Ⅳ</td><td>0.48%</td></tr>
</table>
</details>

<details>
<summary>0825 티어 분포도</summary>
<div align="left"><img src="https://github.com/ho0116/lol_project/assets/85285367/8ca2317e-e720-4b94-aa2b-434c6be4ad34"> </div>
</details>

<details>
<summary>0915 티어 분포도</summary>
<div align="left"><img src="https://github.com/ho0116/lol_project/assets/85285367/38e1a7c6-02fe-4ae7-b95c-c94bf1c329ee"> </div>
</details>

<details>
<summary>0917 티어 분포도</summary>
<div align="left"><img src="https://github.com/ho0116/lol_project/assets/85285367/0a2208ea-e9aa-4b19-8891-315f61aa162f"></div>
</details>


|  |0825|0915|0917|
|:---:|:---:|:---:|:---:|
|플래티넘 ↑|44962|28209|25208|

도출된 플래티넘 이상의 경기에 대해서, 데이터 속성으로 kda, dpm, dpg, dpd, dtpm을 추출했다. 그 이유는 게임의 전반적인 흐름과 전략적인 측면보다는 전투관련지표를 중점으로 삼아 승패 예측을 해보고 싶기때문이다. 그리고 승패를 예측하는 것이기 때문에, 경기 데이터를 기준으로 데이터 속성별 정규화(normalize)를 수행했다. 

### 2.4 데이터 프레임 설계

탐색적 데이터 분석과 데이터 전처리를 통해 다음과 같은 데이터 프레임을 만들고자 한다.
<table>
  <tr align="center"><th>Id</th><th>팀</th><th>주요지표</th><th>TOP</th><th>MID</th><th>JUG</th><th>ADC</th><th>SPT</th></tr>
  <tr align="center"><th rowspan="7">고유번호</th><th rowspan="7">100/200</th><td>kda</td><td>···</td><td>···</td><td>···</td><td>···</td><td>···</td></tr>
  <tr align="center"><td>dpd</td><td>···</td><td>···</td><td>···</td><td>···</td><td>···</td></tr>
  <tr align="center"><td>dpm</td><td>···</td><td>···</td><td>···</td><td>···</td><td>···</td></tr>
  <tr align="center"><td>dpg</td><td>···</td><td>···</td><td>···</td><td>···</td><td>···</td></tr>
  <tr align="center"><td>dtpm</td><td>···</td><td>···</td><td>···</td><td>···</td><td>···</td></tr>
  <tr align="center"><td>win</td><td>···</td><td>···</td><td>···</td><td>···</td><td>···</td></tr>
  <tr align="center"><td>tier</td><td>···</td><td>···</td><td>···</td><td>···</td><td>···</td></tr>
</table>

## 3. 승패예측 모델
### 3.1 모델 개요
각 팀의 특징 데이터를 합산하고 정규화한 후, 승패를 예측하는 단순한 합산을 사용하는 모델이다.
### 3.2 성능
<table>
  <tr align="center"><th></th><th></th><th>정확도(단순)</th></tr>
  <tr align="center"><th rowspan="2">0825</th><td>플래티넘 ↑</td><td>95%</td></tr>
  <tr align="center"><td>전체</td><td>93.6%</td></tr>
  <tr align="center"><th rowspan="2">0915</th><td>플래티넘 ↑</td><td>93.4%</td></tr>
  <tr align="center"><td>전체</td><td>90.3%</td></tr>
  <tr align="center"><th rowspan="2">0917</th><td>플래티넘 ↑</td><td>88.4%</td></tr>
  <tr align="center"><td>전체</td><td>86.2%</td></tr>
</table>

### 3.3 소결
* 성능에 대한 의미
* 핵심 데이터 항목에 대한 추정 및 분석
## 4. 결론 및 배운점
단순한 합산 방법을 활용한 승패 예측 모델은 특정한 날짜에 대해서는 높은 정확도를 보여주고 있지만, 전체적인 정확도에 대한 안정성은 여전히 고려해야 한다. 이러한 한계 때문에 더 정교하게 예측하기 위해서는 CNN과 같은 더 복잡하고 정교한 모델을 사용하는 것이 효과적일 것으로 판단된다. 롤에 대해 관심이 있어서 배경지식이 많았다면 조금 더 수월하지않았을까하는 생각이 든다. 
