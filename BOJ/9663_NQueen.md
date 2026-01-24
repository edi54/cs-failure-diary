# [BOJ 9663] N-Queen — 비트마스크 DFS 오답노트

## 1. 문제 개요

- 문제 번호: BOJ 9663
- 문제명: N-Queen
- 조건: `N ≤ 14`
- 목표: N×N 체스판에 퀸 N개를 서로 공격하지 않게 배치하는 경우의 수 계산


## 2. 최초 오답 접근 요약

- 비트마스크 기반 DFS 사용
- 열(`cols`), 대각선(`diag1`, `diag2`) 충돌 여부를 비트 연산으로 관리
- **row(행) 상태를 명시적으로 두지 않음**


## 3. 오답 원인 분석

### (1) 종료 조건에서 정답 카운팅 누락

- `cols == mask` 상태는 퀸을 N개 모두 배치한 **정답 상태**
- 그러나 종료 시 `ans += 1` 없이 return 처리
- 결과적으로 모든 해를 탐색해도 정답 개수는 0

### (2) DFS 상태 정의 불완전

- N-Queen은 본질적으로 “각 행에 하나의 퀸”을 배치하는 문제
- row 상태가 없으면:
  - 현재 깊이가 몇 번째 행인지 추론해야 함
  - 종료 조건과 디버깅이 어려워짐
- 실수 발생 확률이 높아짐


## 4. 정답 코드에서의 핵심 수정 사항

- DFS 상태를 명확히 정의  
  `(row, cols, diag1, diag2)`
- 종료 조건을 `row == n`으로 설정
- 종료 시 `ans += 1` 명시
- 대각선 비트 shift 이후 `& mask`로 범위 제한


## 5. 최종 정답 코드

```python
import sys
sys.setrecursionlimit(10**7)

def main():
    n = int(sys.stdin.readline().strip())
    mask = (1 << n) - 1
    ans = 0

    def dfs(row, cols, diag1, diag2):
        nonlocal ans
        if row == n:
            ans += 1
            return

        available = mask & ~(cols | diag1 | diag2)
        while available:
            pick = available & -available
            available -= pick
            dfs(
                row + 1,
                cols | pick,
                ((diag1 | pick) << 1) & mask,
                (diag2 | pick) >> 1
            )

    dfs(0, 0, 0, 0)
    print(ans)

if __name__ == "__main__":
    main()
```

## 6. 재발 방지 체크리스트

- 종료 조건에서 **정답 카운팅(`ans += 1`)이 반드시 수행되는지 확인**
- DFS 상태가 문제 정의를 **완전히 포함하는지 검증**
  - N-Queen에서는 `row`를 명시적으로 두는 것이 가장 안전
- 비트 shift 연산 후 **범위 마스킹(`& mask`)이 필요한지 확인**
- 작은 입력값 `N = 1 ~ 6`에 대해
  - 알려진 정답과 결과를 비교하는 **sanity check 수행**


## 7. 회고

- 비트마스크 최적화는 코드 길이를 줄이기 위한 기법이 아니라  
  **상태를 더 엄밀하게 관리하기 위한 도구**
- 상태 정의가 불완전하면, 성능 이전에 **정확성이 무너진다**
- 앞으로의 백트래킹/탐색 문제에서는  
  “상태 → 종료 조건 → 카운팅”을 먼저 설계한 뒤 구현