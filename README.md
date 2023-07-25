# stock-example
[인프런강의 - 재고시스템으로 알아보는 동시성이슈 해결방법 소스입니다.](https://inf.run/Jhu5)


## java synchronized 문제점
- 서버가 1대일때는 되는듯싶으나 여러대의 서버를 사용하게되면 사용하지 않았을때와 동일한 문제가 발생된다.
- 인스턴스단위로 thread-safe 이 보장이 되고, 여러서버가 된다면 여러개의 인스턴스가 있는것과 동일기때문입니다.


## Mysql 을 활용한 다양한 방법
### Pessimistic Lock  
- 실제로 데이터에 Lock 을 걸어서 정합성을 맞추는 방법입니다. 
- exclusive lock 을 걸게되며 다른 트랜잭션에서는 lock 이 해제되기전에 데이터를 가져갈 수 없게됩니다.
- 데드락이 걸릴 수 있기때문에 주의하여 사용하여야 합니다.

### Optimistic Lock  
- 실제로 Lock 을 이용하지 않고 버전을 이용함으로써 정합성을 맞추는 방법입니다. 
-먼저 데이터를 읽은 후에 update 를 수행할 때 현재 내가 읽은 버전이 맞는지 확인하며 업데이트 합니다. 
- 내가 읽은 버전에서 수정사항이 생겼을 경우에는 application에서 다시 읽은후에 작업을 수행해야 합니다.

### Named Lock  
- 이름을 가진 metadata locking 입니다. 
- 이름을 가진 lock 을 획득한 후 해제할때까지 다른 세션은 이 lock 을 획득할 수 없도록 합니다. 
- 주의할점으로는 transaction 이 종료될 때 lock 이 자동으로 해제되지 않습니다. 
- 별도의 명령어로 해제를 수행해주거나 선점시간이 끝나야 해제됩니다.

```
참고
https://dev.mysql.com/doc/refman/8.0/en/
https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html
https://dev.mysql.com/doc/refman/8.0/en/metadata-locking.html
```


## Redis 활용한 다양한 방법

### Lettuce
- 구현이 간단하다
- spring data redis 를 이용하면 lettuce 가 기본이기때문에 별도의 라이브러리를 사용하지 않아도 된다.
- spin lock 방식이기때문에 동시에 많은 스레드가 lock 획득 대기 상태라면 redis 에 부하가 갈 수 있다.

### Redisson
- 락 획득 재시도를 기본으로 제공한다.
- pub-sub 방식으로 구현이 되어있기 때문에 lettuce 와 비교했을 때 redis 에 부하가 덜 간다.
- 별도의 라이브러리를 사용해야한다.
- lock 을 라이브러리 차원에서 제공해주기 떄문에 사용법을 공부해야 한다.

```text
실무에서는 ?
재시도가 필요하지 않은 lock 은 lettuce 활용
재시도가 필요한 경우에는 redisson 를 활용
```

## Mysql VS Redis

### Mysql
- 이미 Mysql 을 사용하고 있다면 별도의 비용없이 사용가능하다.
- 어느정도의 트래픽까지는 문제없이 활용이 가능하다.
- Redis 보다는 성능이 좋지않다.

### Redis
- 활용중인 Redis 가 없다면 별도의 구축비용과 인프라 관리비용이 발생한다.
- Mysql 보다 성능이 좋다.


