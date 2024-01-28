---
title: "Leetcode - k-inverse-pairs-array"
description: "DP의 이유, 캐시 최적화, sliding window"
time: 2024-01-28 18:52:53
tags:
  - coding-interview
---

## sliding-window 구현 방법

sliding-window는 전체 데이터를 일정한 크기로 전체 경우의 수를 처리하는 기법으로 분석된다.

아래는 sliding-window의 사용 방법이다.


``` python
# process_function은 일정한 크기의 arr를 처리하는 함수이다.
def sliding_sum(data_arr: List[int], window_size:int , process_function ):
    total = 0
    for data_idx in range(window_size, data_arr):
        if data_idx - window_size >= 0
            # window_size를 넘은 데이터를 뺀다.
            total = total - data_arr[data_idx - window_size]
        # 전체 데이터에 새로운 idx 데이터를 추가한다.
        total = total + data_arr[data_idx]
```

## dynamic-programming with cache optimization

2d DP를 1d DP로 바꿀 수 있는 방법이다. 이 방법은 구하고자 하는 값이 이전 결과값만 상관있을 때 사용될 수 있다.


``` python
def dp_with_cache_optimization():
    # 처음 시작할 DP 초기화
    previous_cache = [0] * (k + 1)
    for N in range(n):
        # 현재 DP 처리
        current_cache = [0] * (k + 1)
    # 이전 값을 덮어씌운다
    privous_cache = current_cache
```

## DP 판단 방법

dynamic programming을 하기 위해선 현재의 선택이 다음 선택에 영향을 받지 않도록 문제를 쪼개는 것이다.

(3, 1)의 선택지는 1,2,3이 있고 첫 번째 선택한 item을 각각 1, 2, 3으로 했을 때 각 선택지들의 다음 선택지 즉 아래와 같은 선택을 하였을때 남는 item을 기반으로 문제를 다시 쪼갤 수 있는지 확인해야 한다.

```
1 - 2,3
2 - 1,3
3 - 1,2
```

1을 선택했을 때는 2 또는 3이 다음 pair로 나왔을 때, inverse-pair를 만족하지 않으므로 k의 개수는 줄어들지 않는다 즉, 나머지 item인 2,3을 가지고 (2, 1)의 결과 값을 만들어내야 한다.

2는 1이나오면 inverse-pair가 1개 감소한다. 3이 나왔으면 감소하지 않는다. 즉, inverse-pair를 만들 수 있는 경우의 수는 1개이므로 1,3을 가지고 만들 수 있는 2,0의 경우의 수를 구하면 된다.

3은 모두다 inverse-pair이므로 2가지 경우가 나타난다. 즉 1,2를 가지고 k -2 즉 2,-1을 계산해야 하지만, k는 >0이므로 답을 구할 수 없다. 즉 경우의 수는 0이다.
 
