---
title: [EDA] 타이타닉 생존자 분석
date: 2023-11-27 00:00:00 +0900
categories: [Bigdata-AI, EDA]
tags: [titanic, eda]
render_with_liquid: false
future: true
---

이번 시간엔 오늘 진행한 토이플젝 한 것 올리려고 합니다(아직 미완성)

```
plt.figure(figsize=(8,6))
msno.matrix(df)
plt.show()
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/1.png)

결측치부터 확인했습니다. age쪽에 좀 있네요 fare는 결측치가 별로 없어서 바로 평균값으로 대치할 예정입니다.

다만 age는 바로 평균을 대입해버리면 그래프를 그렸을 때 평균만 올라가서 원래의 그래프 형태와 많이 달라집니다.

```
df['age'].fillna(df['age'].mean(),inplace=True)

plt.figure(figsize=(8,6))

sns.kdeplot(x=df_base[df_base["age"].notnull()]['age'], color="Blue", shade = True, label="original" )
sns.kdeplot(x=df['age'],color="red", shade = True, label="tuned")

plt.legend()
plt.show()
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/2.png)

이렇게 전체 평균쪽으로만 위로 솟은 그래프가 되었습니다. 그래서 age는 세분화해서 각각의 평균을 구해서 그 그룹의 결측치에 맞는 평균값을 대입해보겠습니다.

```
df['age'].fillna(df.groupby(["sex","pclass"])['age'].transform('mean'),inplace=True)

plt.figure(figsize=(8,6))

sns.kdeplot(x=df_base[df_base["age"].notnull()]['age'], color="Blue", shade = True, label="original" )
sns.kdeplot(x=df['age'],color="red", shade = True, label="tuned")

plt.legend()
plt.show()
```

groupby()와 transform()를 사용하여 각 그룹의 평균값을 넣었습니다. 그리고 그래프를 그렸네요

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/3.png)

확실히 저번 그래프랑 많이 달라진 것을 알 수 있습니다. 전체적인 모양을 유지하고 볼륨이 커졌습니다. 꼭지점 방향도 더 비슷해졌고요! 이제 이 데이터로 분석을 해봅시다.

```
plt.figure()

sns.FacetGrid(df, col = 'pclass', row = 'sex').map(sns.histplot, 'survived')

plt.show()
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/4.png)

좌석 등급과 성별이 생존에 어떤 영향을 주는지 알아봅시다. 일단 등급이 좋을수록 생존률이 올라갔습니다. 하지만 낮을수록 남자에서 생존률이 급격히 떨어지는 것을 알 수 있습니다.

```
def age_make(x):
    if x<=3.0:
        return "baby"
    elif x<=13.:
        return "kid"
    elif x<=18.:
        return "teenage"
    elif x<=60.:
        return "adult"
    else:
        return "old"

df['cat.age']=df['age'].apply(age_make)

plt.figure(figsize=(12,8))

sns.barplot(data=df,x="cat.age",y="survived", hue="sex",order=['baby','kid','teenage','adult','old'])

plt.show()
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/5.png)

이번엔 연령을 5개의 연령대로 나눴습니다. 기준은 에버랜드 입장권..ㅎㅎ; 근데 old를 60부터로 하향조정했습니다. 표본이 없어서.. 여튼 그래서 나온 연령대와 성별을 기준으로 생존률을 알아봤습니다. 여자는 특이한 점이 연령대가 높아질수록 생존률이 올라갔습니다. 반대로 남자는 떨어지죠.

```
plt.figure(figsize=(12,8))

sns.barplot(data=df,x="cat.age",y="survived", hue="pclass",order=['baby','kid','teenage','adult','old'])

plt.show()
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/6.png)

baby와 old연령대는 안그래도 적었었는데 좌석등급으로 또 3개로 나눠서 그런지 각각 그룹의 표본이 부족해보입니다. 따라서 가운데 3개만 보자면 역시 등급이 높을수록 생존률이 올라갔고 kid의 경우엔 1등급과 2등급 좌석을 전부 생존했습니다.

아직 할 게 더 많네요.. 내일 아침까지 마무리하겠습니다!