---
layout: 2025
title: introduction to AI with XOR problem
date: 2025-09-09 23:20:58
tags: [AI]
---

Presented for the first time at 2018
Translated into Korean

{% asset_img 01.png %}

VHPC Lab 은 글쓴이가 재학했던 학교의 IT 연구실입니다.

{% asset_img 02.png %}

.

{% asset_img 03.png %}

.

{% asset_img 04.png %}

인공지능이란 무엇인가
자연적으로 정의되지 않은 지능
반대의 경우, 자연지능
인공지능은 주로 기계를 통해 구현되기 때문에, 기계지능이라고도 불림
지능을 가진다는 것은, 스스로 학습하고 추론하여 결정함을 의미

{% asset_img 05.png %}

초기 인공지능
임의 알고리듬을 사용함
코드가 인공지능의 동작을 정의
하드웨어의 성격을 지님
동작을 수정하는 것이 (비교적) 어려움
제작자에 따라 인공지능 성능이 엇갈림
1980년대의 전문가 시스템이 대표적인 예시

{% asset_img 06.png %}

머신러닝이란
인공지능 구현 방법론 중 하나
concept: 데이터가 인공지능의 동작을 정의
소프트웨어의 성격을 지님
동작을 수정하는 것이 (비교적) 쉬움
입력된 데이터와 학습전략에 따라 인공지능 성능이 엇갈림

{% asset_img 07.png %}

뉴런
신경망을 구성
각 뉴런은 신호를 전파함
입력값이 threshold 보다 커질 경우
입출력 : 수상돌기 / 축색돌기 (즉, 신경접합부 ㅡsynapseㅡ)
"뇌를 모방해보면 어떨까"에 대한 영감 ㅡinspiringㅡ

퍼셉트론
뉴런을 모방함
각 퍼셉트론은 값을 전파함
조건을 만족할 때 계산을 수행함
입출력 : 단말
노드와 단말로 구성됨

{% asset_img 08.png %}

인공 신경망이란
자연 신경망을 모방한 망 ㅡnetworkㅡ
다양한 계산들이 내포됨
대부분은 더하기와 곱하기
출력값을 수정하기 용이한 설계
가중치 변수만 조절하면 됨
이러한 구조가 학습 비용을 절감함

{% asset_img 09.png %}

인공 신경망을 활용한 머신러닝
단일 퍼셉트론을 사용한 인공 신경망부터 시작되었음
여러 가지 이유로 효과적이지 않았음
부족한 컴퓨팅 파워
비효율적인 학습 전략
대규모 데이터셋의 부재

{% asset_img 10.png %}

딥러닝이란
머신러닝의 일종으로서, Deep Neural Network (이하, D.N.N.) 를 사용함
(인공) 신경망이 매우 복잡하고 거대할 경우, 이를 D.N.N. 이라 부름
다양한 레이어 유형들이 제공되어 유연함

{% asset_img 11.png %}

왜 딥러닝이 유용한가
여러 가지 문제점들이 해결됨
부족한 컴퓨팅 파워
비효율적인 학습 전략
대규모 데이터셋의 부재

{% asset_img 12.png %}

초기 머신러닝과 딥러닝의 비교
2012년부터, 딥러닝 기반 인공지능이 I.L.S.V.R.C. 기반 인공지능을 이김
2016년부터, 해당 영역은 (딥러닝 기반) 인공지능이 장악해버린 것을 알 수 있음

{% asset_img 13.png %}

더 자세히
초기 머신러닝
어떻게 결과를 도출할지 제작자가 결정
(예컨대, 자동차를 인식하기 위해 둥근 선이 바퀴의 특징이고 각진 선이 차체의 특징이라고 정의)
딥러닝
어떻게 결과를 도출할지조차 인공지능이 결정
(예컨대, 자동차를 인식하기 위한 특징이 무엇인지 알려주지 않음)
딥러닝 같은 방식이 (요즘 들어) 추천됨
물론 장단점은 있음

{% asset_img 14.png %}

(생략)

{% asset_img 15.png %}

.

{% asset_img 16.png %}

Gradient Method 란
선형시스템을 수치계산으로 풀어내는 방법론
그 중에서도, Gradient Descent 를 다룰 것 (인공신경망의 가중치값을 조정하는 데에 사용되는 알고리듬)
다음 상황을 가정
곱하기 연산에 {% katex %} (x = -2, y = 3) {% endkatex %} 두 가지 입력값이 제공됨
출력값을 0 에 가깝게 만들고 싶음
입력값만 수정할 수 있음

{% asset_img 17.png %}

무작위 검색은 어떨까
계산할 때마다 무작위 값을 생성
하지만, 항상 운이 좋을 수는 없음
f (함수 출력값) 는 -6 보다 높아질 수도 있음
하지만 (높아진다 하더라도) 234,281,855 같은 큰 수여도 괜찮을까

{% asset_img 18.png %}

편미분이란
변수가 여러 개인 함수를 미분하는 방법
여러 개의 변수들 중 하나만을 다룸
나머지 변수들은 상수 취급
변수 개수에 상관 없이, 공식은 고정

{% asset_img 19.png %}

Numerical Gradient 란
각 입력값에 대해 편미분을 적용하는 방법
(편미분 값을 계속해서 더함)
최적해 찾는 꽤 좋은 방법
하지만 연산비용이 높음
(단, 정밀한 조작 위해 step size 라는 배수 적용)

{% katex %}
f(x,y) = xy \\ \space \\
\frac{\partial f(x,y)}{\partial x} = \frac{f(x+h,y)-f(x,y)}{h} = \frac{xy + hy - xy}{h} = \frac{hy}{h} = y \\ \space \\
\frac{\partial f(x,y)}{\partial y} = \frac{f(x,y+h)-f(x,y)}{h} = \frac{xy + hx - xy}{h} = \frac{hx}{h} = x \\ \space \\
{% endkatex %}

{% asset_img 20.png %}

편미분 값은 출력값에 대한 (입력값이 가지는) 영향력을 의미
(우리가 해야할 것은) 단지 f 값이 0 을 넘어갈 때 멈추는 것
step size 값이 더 정밀하면 오류값도 적음

{% asset_img 21.png %}

Analytic Gradient 란
Numerical Gradient 의 개선판
계산비용에서 보다 효율적임
미분값을 (매번 계산하지 않고) 고정된 값 사용

{% asset_img 22.png %}

이전 예제에 적용해볼 경우
효율성 때문에, 모든 인공지능 프레임워크에서는 이러한 방법을 사용
아무리 더하기 빼기 곱하기 같은 간단한 계산뿐이라고 해도, 총량에서 수 배 십 수 배 차이가 날 수 있기 때문

{% asset_img 23.png %}

다중연산을 어떻게 푸는가
일반적인 경우, 한 가지 연산만 사용하지 않음
그런데, 다중연산일 경우에는 각 연산들이 서로의 정보를 알 수 없음
(그럼에도 불구하고, 편미분값을 알아야 학습을 할 수 있을 텐데 이를 어떻게 하는가가 관건)
입력값으로 {% katex %} (x = -2, y = 5, z = -4) {% endkatex %}
연산으로 {% katex %} q = x + y, f = q \cdot z = (x + y) \cdot z {% endkatex %} 를 가정

{% asset_img 24.png %}

(앞서 살펴봤던 내용과 마찬가지로) q 에 대한 편미분값은 z 이고, z 에 대한 편미분값은 q
x 에 대한 편미분값은 1 이고, y 에 대한 편미분값은 1
이렇게만 결론지으면 되는 걸까
아니다. 우리가 구한 것은 각 연산에서의 국소적 편미분값들이고, 우리가 원하는 것은 f 값에 대한 x y z 변수들의 편미분값이다
(당연하게도, 우리가 조절할 수 있는 값은 x y z 세 가지 변수뿐이기 때문)
즉, 추가로 {% katex %} \frac{\partial f(q,z)}{\partial x}, \frac{\partial f(q,z)}{\partial y} {% endkatex %} 를 찾아야한다
(방금 {% katex %} \frac{\partial f(q,z)}{\partial z} {% endkatex %} 는 찾았음)

{% asset_img 25.png %}

역전파 ㅡBack Propagationㅡ 란
확인했다시피, 다중연산에서 입력값의 편미분값을 구하는 게 불가능해 보였음
예컨대, f 연산은 x 와 y 에 대해 알 수 있는 방법이 없음
역전파는 이러한 문제를 해결 가능
연쇄규칙 ㅡChain Ruleㅡ 을 사용하면, 연산 너머의 입력값들에 대한 편미분값을 획득 가능
(다시 한 번 예제 규칙을 정리하면 다음과 같음)

{% katex %} x = -2, y = 5, z = -4 \\ {% endkatex %}
{% katex %} q(x, y) = x + y, \space f(q, z) = q \cdot z = (x + y) \cdot z {% endkatex %}

{% asset_img 26.png %}

퍼셉트론 학습 준비 완료
최종 f 값을 바꾸기 위한, x y z 미분값들을 모두 구할 수 있음
(최종 f 값이, 0 에 가까워져 가는 것 확인 가능)
당연하게도, 원한다면 step size 값을 음수 사용하는 것도 고려해볼 수 있음
주로, step size 는 0.01 같은 매우 작은 값이 사용됨
예시에서는, 극적인 변화 위해 0.1 로 사용하였음

2장의 내용은 소스코드로도 확인 및 직접 실행해볼 수 있음
https://github.com/BaeMinCheon/introduction-to-ai/tree/master/Chapter02

{% asset_img 27.png %}
{% asset_img 28.png %}
{% asset_img 29.png %}
{% asset_img 30.png %}
{% asset_img 31.png %}
{% asset_img 32.png %}
{% asset_img 33.png %}
{% asset_img 34.png %}
{% asset_img 35.png %}
{% asset_img 36.png %}
{% asset_img 37.png %}
{% asset_img 38.png %}
{% asset_img 39.png %}
{% asset_img 40.png %}
{% asset_img 41.png %}
{% asset_img 42.png %}
{% asset_img 43.png %}
{% asset_img 44.png %}
{% asset_img 45.png %}
{% asset_img 46.png %}
{% asset_img 47.png %}
{% asset_img 48.png %}
{% asset_img 49.png %}
{% asset_img 50.png %}
{% asset_img 51.png %}
{% asset_img 52.png %}
{% asset_img 53.png %}
{% asset_img 54.png %}
{% asset_img 55.png %}
{% asset_img 56.png %}
{% asset_img 57.png %}
{% asset_img 58.png %}
{% asset_img 59.png %}
{% asset_img 60.png %}
{% asset_img 61.png %}
{% asset_img 62.png %}
{% asset_img 63.png %}
{% asset_img 64.png %}
{% asset_img 65.png %}
{% asset_img 66.png %}
{% asset_img 67.png %}
{% asset_img 68.png %}
{% asset_img 69.png %}
{% asset_img 70.png %}
{% asset_img 71.png %}
{% asset_img 72.png %}
{% asset_img 73.png %}
