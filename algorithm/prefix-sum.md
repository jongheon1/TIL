## 누적합

#### 누적합(Prefix Sum)의 이해
- 배열이나 수열에서 특정 구간의 합을 빠르게 계산하기 위한 기법
- 각 위치까지의 누적된 합을 미리 계산해두고 활용
- 구간 합을 O(1)에 계산 가능

```java
// 1. 일반적인 방식 - 구간합을 매번 계산
int[] arr = {1, 2, 3, 4, 5};
int getSum(int start, int end) {
    int sum = 0;
    for (int i = start; i <= end; i++) {
        sum += arr[i];
    }
    return sum;
}
// 구간 [1,3]의 합 = getSum(1, 3) = 9 (2+3+4)

// 2. 누적합 방식
int[] arr = {1, 2, 3, 4, 5};
int[] prefixSum = new int[arr.length];
prefixSum[0] = arr[0];  // 1
for (int i = 1; i < arr.length; i++) {
    prefixSum[i] = prefixSum[i-1] + arr[i];
}
// prefixSum = [1, 3, 6, 10, 15]
// 구간 [1,3]의 합 = prefixSum[3] - prefixSum[0] = 10 - 1 = 9
```

#### Path Sum III 문제 분석
- 이진 트리에서 특정 합을 만드는 경로의 수를 찾는 문제
- 경로는 반드시 부모에서 자식 방향으로만 진행
- 시작과 끝 지점에 제한이 없음 (루트나 리프노드가 아니어도 됨)

**완전 탐색 접근법**
- 각 노드를 시작점으로 하여 모든 가능한 경로 탐색
- 이중 재귀를 사용: 시작점 선택 + 경로 탐색
```java
class Solution {
    int ret = 0;
    public int pathSum(TreeNode root, int targetSum) {
        if (root != null) go(root, targetSum);
        return ret;
    }
    private void go(TreeNode node, int targetSum) {
        calSum(node, 0, targetSum);
        if (node.left != null) go(node.left, targetSum);
        if (node.right != null) go(node.right, targetSum);
    }
    private void calSum(TreeNode node, long curSum, int targetSum) {
        curSum += node.val;
        if (curSum == targetSum) ret++;
        if (node.left != null) calSum(node.left, curSum, targetSum);
        if (node.right != null) calSum(node.right, curSum, targetSum);
    }
}
```
- 시간복잡도: O(n²), 각 노드에서 다시 하위 경로를 모두 탐색

**누적합을 이용한 최적화**
- 루트에서 현재 노드까지의 경로 합을 계산
- Map을 사용하여 이전 경로들의 누적합 빈도수 저장
- 현재 누적합 - 목표값 = 제거해야 할 이전 경로의 합

```java
class Solution {
    Map<Long, Integer> map = new HashMap<>();
    public int pathSum(TreeNode root, int targetSum) {
        map.put(0L, 1);
        return find(root, targetSum, 0L);
    }
	private int find(TreeNode node, int target, long cur) {
	    if (node == null) return 0;
	    
	    cur += node.val;  // 현재까지의 경로 합
	    
	    // (현재 누적합 - target)이 이전에 나온 적이 있다면,
	    // 그 경로를 제거하면 target을 만족하는 경로가 됨
	    int result = map.getOrDefault(cur - target, 0);
	    
	    // 현재 누적합을 맵에 저장
	    map.put(cur, map.getOrDefault(cur, 0) + 1);
	    
	    // 좌우 자식 탐색
	    result += find(node.left, target, cur) + find(node.right, target, cur);
	    
	    // 백트래킹: 현재 경로 제거
	    map.put(cur, map.get(cur) - 1);
	    
	    return result;
	}
}

```

**최적화 방식의 장점**
- 시간복잡도: O(n), 각 노드를 한 번씩만 방문
- 공간복잡도: O(h), h는 트리의 높이 (재귀 스택)
- Map을 통한 경로 합 관리로 중복 계산 제거

**핵심 포인트**
- 누적합 + HashMap을 활용한 최적화
- 백트래킹을 통한 경로 관리
- 음수 값이 포함될 수 있어 long 타입 사용

**활용 가능한 유사 문제**
- 배열에서의 부분합 문제
- 매트릭스에서의 영역 합 계산
- 연속된 부분 수열의 합 문제

이러한 누적합 기법은 구간 합이나 경로 합과 관련된 다양한 문제에서 시간복잡도를 효과적으로 줄일 수 있는 중요한 알고리즘 기법이다.