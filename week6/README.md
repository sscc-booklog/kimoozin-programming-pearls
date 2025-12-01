# 퀵 정렬(Quicksort) 정리 노트

## 0. 공통 설정

- **정렬 대상 배열**: `x[]` (일부 예제에서는 `a[]`로 표기됨)
- **정렬 구간**: 인덱스 `l` ~ `u` (두 인덱스 모두 포함, 0-based 가정)
- **호출 형태**: `qsortX(l, u)` → `x[l..u]` 구간을 **제자리(in-place)**에서 정렬

---

## 1. `qsort1` – 단순 파티션 + 재귀 퀵소트 (Lomuto 스타일 변형)

가장 기본적인 형태의 구현입니다.

### 1-1. 코드

```c
void qsort1(int l, int u) {
    if (l >= u)
        return;

    int m = l;
    for (int i = l + 1; i <= u; i++) {
        /* invariant: x[l+1..m] < x[l] && x[m+1..i-1] >= x[l] */
        if (x[i] < x[l])
            swap(++m, i);
    }
    swap(l, m);        // 피벗을 최종 위치로 이동

    qsort1(l,   m - 1);  // 피벗보다 작은 구간
    qsort1(m + 1, u);    // 피벗보다 크거나 같은 구간
}
```

### 1-2. 핵심 개념

- **피벗(pivot)**: `x[l]`
- **목표 후조건**: 루프 종료 및 마지막 `swap(l, m)` 이후 아래 조건 만족
    - `x[l..m-1] < x[m]`
    - `x[m+1..u] >= x[m]`
    - `x[m]`은 최종 정렬 위치 확정

### 1-3. 루프 불변식 (Loop Invariant)

반복문 실행 도중 항상 다음 조건이 유지됩니다.

> `x[l+1..m] < x[l]` (피벗보다 작은 값들)  
> `x[m+1..i-1] >= x[l]` (피벗보다 크거나 같은 값들)

### 1-4. 동작 요약

1. `m = l`, `i = l+1`에서 시작.
2. `i`를 `l+1`부터 `u`까지 순회하며:
    - 만약 `x[i] < x[l]`이면, `++m` (작은 구간 확장) 후 `swap(m, i)`로 작은 원소를 앞쪽으로 이동.
3. 루프 종료 후 `swap(l, m)`을 통해 피벗을 자신의 자리(`m`)로 옮김.

---

## 2. `qsort3` – Hoare 파티션 (양쪽 포인터) 방식

양쪽 끝에서 중앙으로 좁혀오는 방식으로, 일반적으로 Lomuto 방식보다 교환 횟수가 적어 효율적입니다.

### 2-1. 코드

```c
void qsort3(int l, int u) {
    if (l >= u)
        return;

    int t = x[l];      // 피벗
    int i = l;
    int j = u + 1;

    while (1) {
        // 왼쪽 → 오른쪽: t 이상을 만날 때까지
        do { i++; } while (i <= u && x[i] < t);

        // 오른쪽 → 왼쪽: t 이하를 만날 때까지
        do { j--; } while (x[j] > t);

        if (i > j)
            break;

        swap(i, j);    // 왼쪽 큰 값 / 오른쪽 작은 값 교환
    }

    swap(l, j);        // 피벗을 최종 위치로 이동

    qsort3(l,   j - 1);  // 왼쪽 구간
    qsort3(j + 1, u);    // 오른쪽 구간
}
```

### 2-2. 포인터 동작

- **`i`**: 왼쪽에서 오른쪽으로 진행하며 `x[i] < t`인 동안 이동. (피벗 이상 `≥ t`를 만나면 멈춤)
- **`j`**: 오른쪽에서 왼쪽으로 진행하며 `x[j] > t`인 동안 이동. (피벗 이하 `≤ t`를 만나면 멈춤)

### 2-3. 분할 로직

- **`i <= j` 동안**: `x[i]`(왼쪽인데 큰 값)와 `x[j]`(오른쪽인데 작은 값)를 `swap`하여 제자리로 보냄.
- **`i > j` 되면**: 왼쪽에는 `≤ t`, 오른쪽에는 `≥ t`가 대략 정리됨. 루프 종료.
- **마지막 `swap(l, j)`**: 피벗을 `j` 위치로 이동.
    - 결과: `x[l..j-1] <= x[j]` / `x[j+1..u] >= x[j]`

---

## 3. `qsort4` – 랜덤 피벗 + Cutoff + Hoare 파티션

실무적으로 성능을 높이고 최악의 경우를 방지하기 위한 최적화 버전입니다.

### 3-1. 코드

```c
void qsort4(int l, int u) {
    // 보통은 (u - l + 1 < cutoff) 로 구현하는 것이 자연스럽다.
    if (u - 1 < cutoff)
        return;

    // [l, u] 범위에서 임의의 인덱스를 피벗으로 선택
    swap(l, randint(l, u));

    int t = x[l];   // 피벗
    int i = l;
    int j = u + 1;

    while (1) {
        do { i++; } while (i <= u && x[i] < t);
        do { j--; } while (x[j] > t);

        if (i > j)
            break;

        int temp = x[i];
        x[i] = x[j];
        x[j] = temp;
    }

    swap(l, j);          // 피벗 최종 위치

    qsort4(l,   j - 1); // 왼쪽
    qsort4(j + 1, u);   // 오른쪽
}
```

### 3-2. 특징

1. **Cutoff 최적화**:
    - 구간 길이가 `cutoff` 미만이면 재귀를 중단하고 `return`.
    - 이후 전체 배열에 대해 삽입 정렬(Insertion Sort)을 수행하여 마무리하는 패턴에 적합.
2. **랜덤 피벗 선택**:
    - `randint(l, u)`로 랜덤 인덱스를 골라 맨 앞(`l`)과 교환.
    - 이미 정렬되었거나 역정렬된 입력에서 O(n²)이 발생하는 것을 방지.
3. **분할 방식**: `qsort3`와 동일한 Hoare 파티션 사용.

---

## 4. 이론 1 – 스택 공간: 평균 O(log n) vs 최악 O(n)

### 4-1. 스택 사용량 분석

퀵 정렬은 재귀 호출마다 스택 프레임 1개를 사용하므로 전체 스택 사용량은 **재귀 깊이**에 비례합니다.

- **평균적인 경우**:
    - 피벗이 중간값 근처라면 n → n/2 → n/4 ...
    - 재귀 깊이 ≈ log₂ n → **공간 복잡도 O(log n)**
- **최악의 경우**:
    - 피벗이 항상 최소/최대값인 경우 (예: 정렬된 배열에서 첫 원소 피벗).
    - 분할이 (1, n-1)로만 일어남.
    - 재귀 깊이 ≈ n → **공간 복잡도 O(n)**

### 4-2. 항상 O(log n) 공간을 보장하는 방법 (Tail Call Elimination)

항상 **더 작은 부분 배열을 먼저 재귀 호출**하고, 큰 쪽은 재귀 대신 **while 루프로 처리**하여 스택 깊이를 제어합니다.

**의사 코드 예시:**

```c
void qsort_logstack(int l, int u) {
    while (l < u) {
        int m = partition(l, u);   // 피벗 자리

        int left_len  = m - l;
        int right_len = u - m;

        if (left_len < right_len) {
            // 왼쪽이 더 짧으면 왼쪽만 재귀 (스택 깊이 최소화)
            qsort_logstack(l, m - 1);
            l = m + 1;      // 오른쪽은 while 루프로 계속 (Tail Call)
        } else {
            // 오른쪽이 더 짧으면 오른쪽만 재귀
            qsort_logstack(m + 1, u);
            u = m - 1;      // 왼쪽은 while 루프로 계속
        }
    }
}
```

---

## 5. 이론 2 – "두꺼운 피벗" 3-way 분할 (< t, = t, > t)

### 5-1. 목표 후조건

배열을 피벗 `t`를 기준으로 3개 구간으로 분할합니다.

1. `a[l .. lt-1] < t`
2. `a[lt .. gt] == t` (피벗과 같은 값들)
3. `a[gt+1 .. u] > t`

### 5-2. 3-way Partition 함수 (Dutch National Flag)

```c
// a[l..u]를 피벗 t 기준으로 3구간으로 나눔.
// 결과: lt, gt를 통해 == t 구간 [lt..gt]를 알려줌.
void partition3(int l, int u, int *plt, int *pgt) {
    int t  = a[l];    // 피벗
    int lt = l;       // < t 구간의 끝
    int i  = l + 1;   // 스캔 인덱스
    int gt = u;       // > t 구간의 시작

    while (i <= gt) {
        if (a[i] < t) {
            swap(&a[lt], &a[i]);
            lt++;
            i++;
        } else if (a[i] > t) {
            swap(&a[i], &a[gt]);
            gt--;
            // i는 그대로 두고, 새로 온 a[i]를 다시 검사
        } else { // a[i] == t
            i++;
        }
    }

    *plt = lt;
    *pgt = gt;
}
```

### 5-3. 퀵 정렬에 적용

```c
void qsort_3way(int l, int u) {
    if (l >= u) return;

    int lt, gt;
    partition3(l, u, &lt, &gt);

    // < t 부분만 정렬
    qsort_3way(l, lt - 1);

    // == t 부분은 건드릴 필요 없음 (이미 정렬된 상태로 간주)

    // > t 부분만 정렬
    qsort_3way(gt + 1, u);
}
```

- **장점**: 중복 원소가 많은 경우 재귀 호출 횟수와 비교 횟수가 획기적으로 줄어듭니다.

---

## 6. 이론 3 – 피벗 선택 전략 (Better Pivot Selection)

단일 원소(첫/끝 값)를 피벗으로 쓰면 정렬된 배열 등에서 성능이 저하되므로, **샘플링**을 통해 중간값에 가까운 피벗을 고릅니다.

### 6-1. Median-of-Three (3개 중간값)

가장 흔한 개선 방법으로 왼쪽, 중간, 오른쪽 값 중 중간값을 선택합니다.

```c
int median3(int l, int u) {
    int m = (l + u) / 2;

    if (a[l] > a[m]) swap(&a[l], &a[m]);
    if (a[m] > a[u]) swap(&a[m], &a[u]);
    if (a[l] > a[m]) swap(&a[l], &a[m]);

    // 이제 a[m]이 세 값 중 가운데 값
    return m;
}

// 사용 시:
// int p = median3(l, u);
// swap(&a[l], &a[p]);
// int t = a[l];
```

### 6-2. 기타 전략

- **Random Sample Median**: `randint`로 임의의 위치 3~5개를 뽑아 그 중 중간값을 사용. 입력 패턴에 덜 민감함.
- **Tukey’s Ninther (Median-of-Nine)**:
    - 배열에서 9개 위치를 골라 3개씩 3그룹으로 나눔.
    - 각 그룹의 Median-of-3를 구함 (총 3개).
    - 그 3개의 Median-of-3를 다시 수행 → 최종 피벗.
    - 아주 큰 배열에서 효과적.
- **Median-of-Medians**:
    - 이론적으로 정확한 중앙값을 O(n)에 구할 수 있음.
    - 항상 최악의 경우에도 O(n log n)을 보장하지만, 상수가 커서 실전 라이브러리보다는 이론적 분석에 주로 쓰임.
