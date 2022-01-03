# Elasticsearch VM Memory Warning Issue
Elasticsearch 관련으로 다음의 에러 메시지가 출력되어 기록을 남기고자 한다.

<div class="notice--info" markdown="1">
📢 OpenJDK 64-Bit Server VM warning: INFO: os::commit_memory(0x000000012b990000, 29333323776, 0) failed; error='Not enough space' (errno=12)
</div>

로그 내용을 보면 금방 알 수 있다시피, **OS 상의 메모리가 부족해서 출력되는 메시지**이다.

이 현상을 해결하기 위해, 보통은 SWAP 영역을 확보해서 OS 메모리가 부족할 때 일시적으로 사용할 수 있도록 조치하는 경우가 일반적이다.

하지만 해당 서버는 Elasticsearch 를 사용하는 서버이고, 보통 Elasticsearch 는 SWAP 을 통해 기동할 경우 퍼포먼스가 상당히 저하되기 때문에 일부러 OS 쪽의 SWAP 사이즈를 잡지 않거나, 영역을 만들더라도 사용하지 않도록 설정해두곤 한다.

이런 상황에서 해당 이슈를 해결하기 위한 방법은 무엇이 있을까?

## jvm.options 의 Heap Size 줄이기

OS 상의 메모리 부족이 문제라면, Elasticsearch 에서 전용으로 사용할 Heap Size 값을 조정하는 것으로 문제를 해결할 수 있다고 한다.

Elasticsearch Heap Size 의 디폴트값은 2GB 로 설정되어 있는데, 애초에 메모리 사이즈가 작은 서버에 Elasticsearch 를 올리거나, 임의적으로 Heap Size 를 크게 잡아둔 상태에서 운영하다가 서버 자체의 남은 Heap Size 가 부족해졌을 경우 위의 에러 메시지가 출력되므로 상황에 따라 Elasticsearch 자체에서 전용으로 사용할 Heap Size 를 줄여주는 것이 해결책이 될 수 있다.

(물론, Elasticsearch 자체에서 사용할 메모리 사이즈가 충분히 확보되지 않아 차후 벌어질 OutOfMemory 이슈를 고려해서 조정해야 한다)

```bash
## Xms 와 Xmx 는 같게 설정한다
## 최대 heap size 와 최소 heap size 를 조정하는 과정에서 일어날 퍼포먼스 저하를 방지하기 위해
-Xms2g ## 이 부분을 조정한다. 1g 나 512m 등
-Xmx2g ## 이 부분을 조정한다. 1g 나 512m 등
```

## `vm.max_map_count` 조정하기

OS 상의 nmap 설정값을 조정하는 것은 Elasticsearch 초기 구축 단계에서 논의되는 바이기는 하나, 만약 이 설정 없이 운영 중에 메모리 부족 상태(정확히는 nmap 수가 적은 데서 오는 이슈)에 빠질 경우에는 해당 수치를 조정해주는 것이 해결 방안이 될 수 있다.

```bash
## 바로 설정값 수정하기
sudo sysctl -w vm.max_map_count=<수정할 값>
## 영구적으로 설정하기
vi /etc/sysctl.conf
vm.map_max_count=<수정할 값>
reboot
```

## 참고

- [https://www.facebook.com/groups/elasticsearch.kr/permalink/1420288481390321/](https://www.facebook.com/groups/elasticsearch.kr/permalink/1420288481390321/)
- [https://stackoverflow.com/questions/34748464/ubuntu-elasticsearch-error-cannot-allocate-memory](https://stackoverflow.com/questions/34748464/ubuntu-elasticsearch-error-cannot-allocate-memory)
- [https://icarus8050.tistory.com/54](https://icarus8050.tistory.com/54)
