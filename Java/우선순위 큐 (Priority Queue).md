
일반적인 큐와는 다르게 데이터를 삽입한 순서와 상관 없이 정의된 우선순위가 높은 순서대로 데이터를 내보내는 자료구조이다.

~~~ java
// 오름차순 정렬
PriorityQueue<Object> pQ = new PriorityQueue<>();  

// 내림차순 정렬
PriorityQueue<Object> reversedPQ = new PriorityQueue<>(Collections.reverseOrder());
~~~

