---
title: "[Redis] 캐싱 전략"
excerpt: ""

categories:
  - Redis
tags:
  - [Redis, Cache]

permalink: /redis/caching-strategies/

toc: true
toc_sticky: true

date: 2024-09-07
last_modified_at: 2024-09-07
---
# 📋 개요
캐싱 전략은 웹 서비스 환경에서 **시스템 성능 향상**을 기대할 수 있는 기술이다.  
이번 포스팅은 [캐시](https://ijnooyah.github.io/redis/cache/){: target='_blank'}, [레디스](https://ijnooyah.github.io/redis/redis/){: target='_blank'}에 대한 이해가 선행되어야 한다.

캐싱 전략을 들어가기 앞서 **cache hit**와 **cache miss** 용어에 대해 아는 것이 필요하다.  

<b>💡 용어</b>  
- **Cache Hit**: 필요한 데이터가 **캐시에 존재**할 때, 이를 Cache Hit라고 한다.  
- **Cache Miss**: 필요한 데이터가 **캐시에 없을 때**, 이를 Cache Miss라고 한다.

---
# ⚖️ 캐싱의 주요 과제: 데이터 정합성
캐시를 사용할 때 가장 빈번하게 발생하는 문제는 **데이터 정합성 문제**이다.  
데이터 정합성이란, 어느 한 데이터가 캐시(**Cache Store**)와 DB(**Data Store**) 이 두 곳에서 같은 데이터임에도 불구하고 데이터 정보값이 서로 다른 현상을 말한다. 예를 들어, **DB에서 최신 정보가 업데이트**되었지만 **캐시에 그 정보가 반영되지 않은** 경우가 여기에 해당된다.

따라서 적절한 **캐시 읽기 전략**(Read Cache Strategy)과 **캐시 쓰기 전략**(Write Cache Strategy)을 통해 캐시와 DB간의 **데이터 불일치 문제**를 최소화하면서도 **성능을 최적화**해야 한다.

---

## 👀 캐시 읽기 전략
<b>Look Aside(Cache Aside) 패턴</b>
- 데이터 조회 시 먼저 **캐시를 확인**하고, 없으면 **DB에서 조회**
- 장점:
  - **반복적인 읽기 작업에 효율적**
  - **캐시 장애 시에도 서비스 지속 가능**
- 단점:
  - 캐시 장애 시 **DB에 부하 집중** 가능성
  - 초기 조회 시 항상 DB 호출 필요해 **단건 호출 빈도가 높은 서비스에는 부적합**

![Look Aside 패턴](/assets/images/posts_img/redis/caching-strategies/look-aside.drawio.png)

1. **Cache Store**에 검색하는 데이터가 **있는지 확인** (**cache hit 시도**)
2. **Cache Store에 없을 경우 DB에서 데이터 조회** (**cache miss**)
3. DB에서 조회해온 데이터를 **Cache Store에 업데이트**

이 패턴은 일반적으로 **가장 널리 사용되는 기본 전략**이다.

또한, Look Aside 패턴에서는 DB에서 미리 데이터를 캐시에 넣어두는 작업을 하는데, 이를 **Cache Warming**이라고 한다.

> **Cache Warming 란?**  
서비스 초기에 DB의 데이터를 미리 캐시로 넣어두는 작업이다.  
이렇게 함으로써 초기 트랙픽 급증 시 **대량의 cache miss로 인한 DB 부하를 방지**할 수 있다.
캐시의 제한된 용량과 TTL(Time To Live) 설정에 따라 캐시된 데이터가 만료되기 때문에 **지속적인 관리와 모니터링이 필요**하다.
{: .info}

> **TTL(Time To Live) 란?**  
캐시나 네트워크 패킷에서 데이터를 얼마나 오랫동안 유지할지를 결정하는 시간 설정을 말한다. 주로 캐시 시스템에서 사용되는 TTL은 특정 데이터가 캐시에 저장된 후, 얼마 동안 유효한 상태로 유지될지를 정의하는 값이다.
{: .info}

---

<b>Read Through 패턴</b>
- **캐시에서만 데이터를 읽고**(inline cache), 데이터를 캐시 제공자에게 **위임하는 전략** 
- **캐시 저장 주체**가 Sever가 아닌 **Data Store**
- 장점: 
  - 캐시와 DB 간의 **데이터 동기화가 보장**되어 **데이터 정합성 문제 해결**
  - **읽기가 많은 워크로드에 적합**
- 단점:
  - Look Asied보다 전체 조회 속도가 느릴 수 있음
  - **캐시 장애 시 서비스 전체에 영향을 줄 수 있음** (캐시에 전적으로 의존)

![Read Through 패턴](/assets/images/posts_img/redis/caching-strategies/read-through.png)

1. **Cache Store**에 검색하는 데이터가 **있는지 확인** (**cache hit 시도**)
2. Cache Store에 없을 경우 DB에서 데이터를 조회해 **자체(캐시) 업데이트** (**cache miss**)
3. **업데이트 된 데이터 서버에 반환**

이 패턴은 캐시 미스가 발생하면 캐시가 DB에서 데이터를 가져와 저장한다. 이는 직접적인 DB 접근을 줄이지만, cache miss 시 **내부적으로 DB 조회 및 캐시 저장이 동시에 이루어져서 오버헤드**가 발생할 수 있다. 이 때문에 대규모 서비스에서는 캐시 장애를 방지하기 위해 **Replication** 또는 **Cluster** 구성을 통해 가용성을 높여야 한다.

> **Replicationd와 Cluster**  
Replication은 데이터를 여러 서버에 복제하는 방식이고, Cluster는 데이터를 여러 노드에 분산 저장하는 방식이다. 각 노드는 전체 데이터의 일부를 저장한다.  
Replication은 주로 데이터의 안정성과 읽기 성능 향상에 중점을 두는 반면, Cluster는 대규모 데이터 처리와 시스템의 확장성에 초점을 맞춘다.
{: .info}

> 이 방식 또한 서비스 운영 초반에 Cache Warming을 수행하는 것이 좋다.

---

## ✍️ 캐시 쓰기 전략
<b>Write Back(Write Behind) 패턴</b>
- 캐시와 DB 동기화를 **비동기적으로 처리** (데이터 변경 시점에 **즉시 DB 업데이트를 안함**)
- 데이터를 저장할 때 DB에 바로 쿼리하지않고, 캐시에 모아서 일정 주기 배치 작업을 통해 DB에 반영
- 장점:
  - **쓰기 쿼리 횟수와 DB 부하를 줄일 수 있음**
  - **빈번한 쓰기 작업이 필요한 서비스에 적합**
- 단점
  - **캐시 장애 시 데이터 영구 소실 위험**
  - 자주 사용되지 않는 데이터도 저장되어 **리소스 낭비 가능성**

![Write Back 패턴](/assets/images/posts_img/redis/caching-strategies/write-back.drawio.png)

1. 모든 데이터를 Cache Store에 저장
2. **일정 시간 후 DB에 일괄 저장**

> 이 패턴에서 캐시 장애가 발생하거나 시스템이 비정상 종료될 경우, 아직 DB에 저장되지 않은 데이터는 **영구적으로 손실될 위험**이 있다. 따라서 이 패턴을 사용할 때 역시 **Replication이나 Cluster 구성**을 통해 장애 대응을 해야 하며, 장애 발생 시에도 데이터가 유실되지 않도록 대비하는 것이 중요하다.

---

<b>Write Through 패턴</b>
- 데이터를 **캐시와 DB에 동시에 저장**
- 장점:
  - 캐시의 데이터가 **항상 최신 상태로 유지**되어 **일관성 보장**
  - **데이터 유실이 발생하면 안되는 상황에 적합**
- 단점:
  - **매 요청마다 캐시와 DB에 모두 쓰기 작업**을 하므로 **성능 저하 발생 가능**

![Write Through 패턴](/assets/images/posts_img/redis/caching-strategies/writre-through.drawio.png)

1. 데이터를 Cache Store에 저장
2. **즉시** Cache Store에서 DB로 데이터 저장

---

<b>Write Around 패턴</b>
- **모든 데이터는 DB에 저장**
- **cache miss가 발생하는 경우**에만 **DB와 캐시에 데이터를 저장**
- 장점:
  - Write Through보다 **훨씬 빠른 쓰기 성능**
- 단점:
   - 캐시와 DB 내의 데이터가 **불일치할 수 있음**

![Write Around 패턴](/assets/images/posts_img/redis/caching-strategies/writre-around.drawio.png)

Write Around 패턴은 Write Thorugh보다 빠르지만, cache miss 발생 전에 DB 데이터가 수정되면 캐시와 DB 간 데이터 불일치가 발생할 수 있다. 이를 방지하기 위해 DB 데이터 수정/삭제 시 캐시도 함께 갱신하거나, 캐시의 TTL을 짧게 설정하는 등의 대책이 필요하다.

> Write Around 패턴은 주로 Look Aside, Read Through와 결합해서 사용된다  
데이터가 한 번 쓰여지고, 덜 자주 읽히거나 읽지 않는 상황에서 좋은 성능을 제공한다.

---

## 🤝 캐시 읽기 + 쓰기 전략 조합
<b>Look Aside + Write Around 조합</b>

![Look Aside + Write Around 조합](/assets/images/posts_img/redis/caching-strategies/lookaside_writearound.drawio.png)

가장 일반적으로 자주 쓰이는 조합이다.  
읽기 시 캐시를 먼저 확인하고, 쓰기는 DB에 직접 수행함으로써 읽기와 쓰기 성능의 균형을 맞춘다.

---

<b>Read Through + Write Around 조합</b>

![Read Through + Write Around 조합](/assets/images/posts_img/redis/caching-strategies/readthrough_writearound.drawio.png)

항상 DB에 쓰고, 캐시에서 읽을 때 항상 DB를 거치므로 데이터 정합성 이슈에 대한 강력한 안전 장치를 제공한다.

---

<b>Read Thorugh + Write Through 조합</b>

![Read Thorugh + Write Through 조합](/assets/images/posts_img/redis/caching-strategies/readthrough_writethrough.drawio.png)

데이터를 쓸 때 항상 캐시에 먼저 쓰므로 읽기 시 최신 데이터를 보장하며, 동시에 DB와의 동기화도 즉시 이루어져 데이터 정합성을 높은 수준으로 유지할 수 있다. 

---

# 🧠 상황에 따른 전략 선택
- **자주 읽히고, 변경이 적은 데이터** 
  - Read Thorugh 패턴
  - cache hit rate(캐시 히트율)가 높아 성능 향상이 크고, 데이터 일관성도 유지하기 쉽다.
- **자주 변경되는 데이터**
  - Write Through 패턴
  - 데이터 변경 시 즉시 캐시와 DB가 동기화되어 항상 최신 데이터를 제공할 수 있다ㅣ.
- **대용량 데이터** 
  - Write Back 패턴
  - DB에 대한 쓰기 빈도를 줄여 DB 부하를 완화할 수 있다. 하지만 데이터 손실 위험을 고려해야 한다.

---

# 💽 캐시 저장 방식 지침

캐시 솔루션은 **자주 사용되면서 자주 변경되지 않는 데이터**를 캐시 서버에 적용하면 **Cache Hit Rate**가 높아져 **성능 향상**을 이룰 수 있다. 

특히 메모리 기반 캐시 시스템인 Redis나 Memcached는 빠른 데이터 접근을 제공하지만, **저장 공간이 제한적이므로 데이터 선정과 관리를 잘해야 한다.**

<b>주요 지침</b>
1. **자주 사용되며 자주 변경되지 않는 데이터 캐싱:** 트래픽이 집중되는 데이터는 캐시에 저장하여 빠른 조회 성능을 확보한다. 이때, 캐시 공간은 제한적이므로 파레토 법칙(8:2)을 적용해 20%의 중요한 데이터를 캐시하는 전략이 효과적이다.

2. **휘발성 고려:** 캐시 솔루션은 휘발성을 기본으로 하기 때문에, 데이터 유실 가능성을 염두에 두고 **중요한 데이터는 캐시에만 의존하지 않도록** 해야 한다. 주기적으로 데이터를 디스크에 저장하거나 DB와 동기화하는 방식이 필요하다.

3. **디스크와의 균형:** 실시간 디스크 저장은 성능 저하를 유발할 수 있으므로, 비동기적으로 주기적인 백업을 진행해 성능과 데이터 보존의 균형을 맞춰야 한다.

---

# 🗑️ 캐시 제거 방식 지침

캐시는 제한된 공간을 효율적으로 사용하기 위해 **적절한 만료 정책과 제거 정책**이 필수적이다. 만료 정책이 없으면, 클라이언트는 오래된 데이터를 사용할 위험이 생긴다. 반대로 너무 빠른 제거는 캐시의 이점을 잃게 만든다.

<b>주요 지침</b>
1. **적절한 만료 주기 설정:** 만료 주기가 너무 짧으면 캐시의 장점이 사라지고, 너무 길면 메모리 부족이나 데이터 불일치 문제가 발생할 수 있다. **서비스 특성에 맞는 주기를 설정해 데이터의 최신성과 성능을 유지**해야 한다.
2. **캐시와 DB 동기화:** 캐시 데이터는 보통 영구 저장소(DB)의 복사본이므로, 두 시스템 간 데이터 동기화가 반드시 필요하다. DB와 캐시의 commit 시점을 고려해 데이터 불일치를 방지해야 한다.
3. **Cache Stampede 방지:** 대규모 트래픽 환경에서 TTL이 짧게 설정된 경우, 여러 서버가 동시에 DB에 접근하는 Cache Stampede 현상이 발생할 수 있다. 이를 방지하기 위해 TTL을 조정하거나, 캐시 갱신 시 분산 락을 적용하는 방법을 고려해야 한다.

> **분산 락이란?**
분산 환경에서 여러 프로세스나 스레드가 공유 자원에 동시에 접근하려 할 때 발생하는 경쟁 상태를 방지하기 위해 사용하는 기술이다. 특히, 캐시 시스템에서 캐시 갱신 시 여러 클라이언트가 동시에 cache miss하고 DB에 접근하는 상황을 방지하기 위해 활용된다.
{: .info}

---

# 🌐 캐시 공유 방식 지침

캐시는 애플리케이션의 여러 인스턴스에서 공유되므로 데이터 정합성 유지가 중요한 과제이다. 각 인스턴스가 캐시를 동시에 수정할 수 있기 때문에, **충돌 방지 전략을 적용**해야 한다.

<b>주요 지침</b>  
1. **데이터 충돌방지:** 캐시 데이터를 수정하기 전, 다른 인스턴스가 데이터를 변경했는지 확인해야 한다. 확인 후 변경되지 않았다면 즉시 업데이트하고, 충돌이 발생할 경우 **애플리케이션 레벨**에서 처리하도록 설계한다.
2. **Lock 사용:** 빈번한 업데이트가 발생하는 상황에서는 캐시 데이터를 업데이트할 때 잠금(Lock)을 사용하는 방법도 고려할 수 있다. 이 방법은 데이터 충돌을 방지하지만, 조회성 업무에서는 성능 저하가 발생할 수 있으므로 **신중하게 적용**해야 한다.

---

<p class="ref">참고</p>
- [https://inpa.tistory.com/entry/REDIS-📚-캐시Cache-설계-전략-지침-총정리](https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-%EC%BA%90%EC%8B%9CCache-%EC%84%A4%EA%B3%84-%EC%A0%84%EB%9E%B5-%EC%A7%80%EC%B9%A8-%EC%B4%9D%EC%A0%95%EB%A6%AC){: target="_blank"}

