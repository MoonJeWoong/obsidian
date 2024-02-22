
Disjoint Set (서로소) 집합들을 만들어내는 방법.
Union 과 Find 메서드를 구현한다.

Union : 주어지는 매개변수 값들을 같은 집합으로 만들어라.
Find : 주어지는 매개변수가 몇 번 집합에 속하는 값인지 찾아라.

초기화 방식 팁!
- 문제에서 주어지는 각 요소들마다 하나씩 개별적으로 집합을 가지는 것으로 초기화한다고 생각하자.

~~~ java

private static int[] studentGroups;  
  
private static int find(int value) {  
    if (value == studentGroups[value]) return value;  
    return studentGroups[value] = find(studentGroups[value]);  
}  
  
private static void union(int a, int b) {  
    int groupOfA = find(a);  
    int groupOfB = find(b);  
    if (groupOfA != groupOfB) studentGroups[groupOfA] = groupOfB;  
}

~~~