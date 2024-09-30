---
title: Ensemble Learning Overview
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-09-28 18:23:00 +0900
categories: [ML, EnsembleLearning]
tags: [ML, EnsembleLearning]
pin: false
math: false
---

## What is Ensemble Learning?

다양한 알고리듬(algorithm)을 결합하여, 개별 알고리듬보다 더 우수한 성능을 만들어내는 방법입니다.  
이 자체가 알고리듬은 아니고, 섞는(ensemble) 방법에 관한 얘기입니다.  

## Background(탄생 배경)

### [No Free Lunch Theorem](https://en.wikipedia.org/wiki/No_free_lunch_theorem)(공짜 점심 같은건 없다.)

> Every algorithm scored best or next-to-best on at least two of the six data sets.
>

‘No Free Lunch Theorem’는 ‘success(성공)으로 가는 지름길은 없다, 쉬운길은 없다.’는 의미를 담고 있는 수학계에서 전해지는 말입니다.  
‘Ensemble Learning’에선, **‘모든 문제에 대해서 최고의 결과물을 내는 하나의 알고리듬은 없다’**를 의미합니다.  
때문에, **상황에 맞춰서 알고리듬을 선택하는것이 중요했습니다.**

![여러 알고리듬이 있지만, 모든 dataset에서 최고의 성적을 기록하는 알고리듬은 없습니다.](/assets/img/for-post/Ensemble Learning Overview/image.png)
_여러 알고리듬이 있지만, 모든 dataset에서 최고의 성적을 기록하는 알고리듬은 없습니다._

### Motivation

지금까지는, 상황에 맞는 알고리듬을 선택하는데 집중해 왔지만, 이 논문에선 아래와 같은 얘기를 하고 있습니다.

> 여러 알고리듬을 섞은(ensemble)한 결과가, 단일 알고리듬중 최고의 결과보다 더 좋다.
>

> 이는 실험적으로 발견된 현상이라고 합니다.
{: .prompt-tip }


![Ensemble 기법인 average, vote..등 상관 없이 단일 모델보다 더 개선된 모습을 보여줍니다.](/assets/img/for-post/Ensemble Learning Overview/image%201.png)
_Ensemble 기법인 average, vote..등 상관 없이 단일 모델보다 더 개선된 모습을 보여줍니다._

## 실험 결과 및 Conclusion(결론)

실험 결과, ‘Ensemble Learning’에 해당하는, ‘RF(Random Forest)’와 ‘BST-DT’와 같이 ensemble한 형태가 단일 모델 보다 모든 dataset에 대해 더 나은 결과를 내었습니다.  

![image.png](/assets/img/for-post/Ensemble Learning Overview/image%202.png)

## References
고려대학교 일반대학원 산업경영공학과 강의
: [04-1: Ensemble Learning - Overview](https://youtu.be/1OEeguDBsLU?si=DbW-6Hj0A2Z3tgiI)

**Ensemble Methods in Data Mining paper**
: [Ensemble Methods in Data Mining](https://link.springer.com/book/10.1007/978-3-031-01899-2)
: [Ensemble Methods in Data Mining](https://books.google.co.kr/books?id=DXZjAQAAQBAJ&lpg=PR13&ots=em1PRM964T&dq=Ensemble%20Methods%20in%20Data%20Mining&lr&hl=ko&pg=PR1#v=onepage&q&f=false)

**Paper: Do we Need Hundreds of Classifiers to Solve Real World Classification Problems?**
: ‘Ensemble Learning’을 엄청나게 많은 ‘classifier’에 대해 적용해서 이론을 확인하는 논문.
: [jmlr.org](https://jmlr.org/papers/volume15/delgado14a/delgado14a.pdf)
