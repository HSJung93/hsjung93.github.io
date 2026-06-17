---
title: "컴파일 과정에서 살아난 좀비 GIL 추적기"
date: 2026-06-17 08:00:00 +0900
categories: [개발, 성능]
tags: [Cython, GIL, 성능, 오픈소스, 인과추론, causalml]
---

업리프트 모델로 마케팅 성과를 측정하기 위하여 통계 패키지들을 찾아서 돌려보고 있었다. 그 중 Uber 깃허브 레포지토리에 올라온 [causalml](https://github.com/uber/causalml)(0.16.0) 패키지의 `CausalRandomForestRegressor` 함수로 모델을 학습하던 중 이상한 점을 발견했다. 느린 속도에 모델 학습을 병렬화하기 위하여 `n_jobs` 파라미터의 값을 올렸더니 오히려 느려졌던 것이다. 4개의 워커로 모델을 학습하기 위하여 `n_jobs=4`로 파라미터를 설정하자 기본값인 `n_jobs=1`일 때보다 약 6배 느려졌다. 병렬 작업이 느려졌던 원인은, Cython 소스 코드에 하드코딩된 `linetrace` 디렉티브가 배포 wheel에까지 프로파일링 훅을 컴파일해 넣었기 때문이었다. 이 글에서는 내가 이 원인을 증상으로부터 어떻게 발견했는지, 그리고 검증 및 수정에 이르기까지의 과정을 정리해 보았다. 참고로 이 과정에서 내가 제기한 이슈와 수정 사항은 이제 Uber causalml의 소스코드에 포함되어 있다.

- Pull request: [uber/causalml#914](https://github.com/uber/causalml/pull/914)
- Issue: [uber/causalml#913](https://github.com/uber/causalml/issues/913)

## 증상

여러 메타러너(T/S-Learner)와 Causal Forest를 비교하던 중 Causal Forest만 비정상적으로 느렸다. 병렬화를 위해 `n_jobs`를 조정하자 직관과 반대되는 결과가 나왔다. 코어가 늘어날수록 오히려 학습에 걸리는 시간이 늘어났던 것이다.

| 설정                     | 학습 시간 |
| ------------------------ | --------: |
| 기본값 (`n_jobs` 미지정) |     29.0s |
| `n_jobs=1`               |     29.5s |
| `n_jobs=4`               |    192.6s |

_PyPI wheel 0.16.0, 50 trees, 12k rows 기준._

학습에 걸리는 시간만 늘어날 뿐 별다른 에러도 경고도 없었다. 이상한 점을 찾아내기 어려워 버그가 남아있기 좋은 환경이었다.

## 원인 분석

코어를 늘릴수록 느려진다는 것은 워커들이 어딘가에서 직렬화되고 있다는 신호였다. 병렬 작업하는 워커들이 하나의 잠금으로 수렴하는 지점을 찾아야 했다.

causalml의 트리 학습은 속도를 위해 Cython으로 작성되어 있다. 그 소스 5개의 `.pyx`(와 대응하는 `.pxd` 헤더)에 아래와 같은 디렉티브가 하드코딩되어 있었다.

```cython
# cython: linetrace=True
```

`linetrace`는 테스트 커버리지 측정에 쓰는 도구다. 문제는 이 설정이 개발용 빌드에 그치지 않고 배포 wheel에까지 컴파일되어 들어간다는 점이었다. 이 디렉티브가 켜지면 CPython용으로 생성되는 C 코드가 `CYTHON_PROFILE 1`로 빌드되고, 모든 함수에 프로파일링 훅(`__Pyx_TraceCall` / `__Pyx_TraceSetupAndCall` 등)이 삽입된다. 여기에는 핫루프인 `nogil` 트리 학습 criterion 함수도 포함된다.

이 훅들 때문에, GIL을 풀고 돌아야 할 워커 스레드들이 훅에서 다시 GIL을 잡으며 직렬화된다. 스레드가 많아질수록 경합이 커지고, 그래서 `n_jobs=4`가 `n_jobs=1` (=기본값)에 비해서 느렸던 것이다.

참고로 배포된 바이너리에 훅이 실제로 들어 있는지는 심볼로 직접 확인하면 된다.

```console
$ nm .../causalml/inference/tree/causal/_criterion.cpython-312-darwin.so \
    | grep -E "TraceSetupAndCall|call_return_trace"
... __Pyx_TraceSetupAndCall ...
... __Pyx_call_return_trace_func ...
```

## 검증과 재현

검증을 위하여 같은 머신·같은 Cython·같은 `setup.py`에서 `linetrace=True` 디렉티브만 달리한 두 빌드를 비교해 보았다. 결과는 아래와 같았다.

| 빌드                        | `.so`의 프로파일 훅 | n_jobs=1 | n_jobs=4 |
| --------------------------- | :-----------------: | -------: | -------: |
| `linetrace=True` 존재(원본) |   있음 (심볼 2개)   |    29.4s |   190.9s |
| `linetrace=True` 제거       |        없음         |     4.1s |     1.7s |


`linetrace=True` 디렉티브를 제거하여 바이너리에서 훅이 사라진 결과, 단일 스레드 학습이 약 7배 빨라지며, `n_jobs`가 다시 가속으로 작동하는 것으로 나타났다(`n_jobs=4`가 가장 느린 행에서 가장 빠른 행으로 바뀐다). 컴파일 방식만 수정한 것이기에 통계 모델의 예측은 변하지 않았다. 다운스트림 T-Learner의 Qini 점수가 소수점 6자리까지 동일했고, `tests/test_causal_trees.py`도 그대로 통과했다.

검증 과정을 독립된 환경에서 간편히 재현할 수 있도록 Docker로 만들었다. Docker 파일 및 재현 명령어는 서론에 언급한 issue에서 확인할 수 있다.

## 조치

디렉티브를 소스에 하드코딩하지 않고 환경 변수로 옵트인하도록 옮겼다. 아래와 같이 `setup.py`에서 `CYTHON_TRACE`가 설정됐을 때만 linetrace를 켜도록 하였다. 

```python
import os
linetrace = os.environ.get("CYTHON_TRACE", "0") not in ("0", "", "false", "False")
define_macros = [("CYTHON_TRACE", "1"), ("CYTHON_TRACE_NOGIL", "1")] if linetrace else []
# Extension(..., define_macros=define_macros)
# cythonize(..., compiler_directives={"linetrace": linetrace})
```

그리고 5개 `.pyx`와 `.pxd` 헤더에서 `# cython: linetrace=True`를 제거했다. 커버리지 빌드는 `CYTHON_TRACE=1 python setup.py build_ext`로 돌리면 계측을 그대로 켤 수 있고, 그 외 배포 빌드는 더 이상 계측되지 않는다. 성능 측정 도구를 없애지는 않았고, 함수 실행 시 포함되지 않도록 빌드 시점으로 분리하였다.

## 정리

이 버그는 에러나 경고 없이 "모델이 무겁다"는 인상으로 위장된다는 점이 까다로웠던 것 같다. 실마리는 `n_jobs`를 늘렸을 때 오히려 학습에 걸리는 시간이 늘어난다는, 직관과는 반대되는 결과였다. 통계 라이브러리를 쓰러 들어갔다가 생성된 C 코드와 빌드 설정까지 내려간 셈인데, 이번 기회에 말로만 듣던 python GIL 과 wheel 파일 컴파일/빌드까지 파고들어가볼 수 있어서 재미있었다. 

모델이 옳은 답을 주는지만큼이나 그 답을 제때 주는지도 중요하다고 생각한다. 느린 도구는 충분히 쓰이지 못하고, 그래서 충분히 검증되지도 못하니까 말이다. 대학원에서 통계학을 공부하며 모델을 돌릴 때에는 성능 개선을 위해서 코드를 살펴볼 생각까지는 하지 못했다. 지난 5년동안 서버 백엔드 엔지니어로 일했던 경험과 습관에 디버깅이 자연스러웠다. 현재는 팀을 옮겨 마케팅 성과를 측정하기 위한 인과추론 프로젝트를 진행 중이다. 오랜만에 다시 통계 모델을 돌리고 있지만, 그 사이 내가 무언가 한가지 넓어졌고 깊어졌음을 느낄 수 있었다는 점에서 반가운 버그였다.
