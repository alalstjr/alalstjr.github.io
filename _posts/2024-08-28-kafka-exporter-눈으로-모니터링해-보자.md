---
title: kafka-exporter 눈으로 모니터링해 보자
description: kafka-exporter 실무에 적용하는 과정을 그대로 공유합니다.
author: JJunpro
date: 2024-08-28 00:00:00 +0800
categories: [ 지식공유 ]
tags: [ kafka,kafka-exporter,helm,모니터링,prometheus,grafana ]
pin: false
math: true
mermaid: true
image:
  path: /assets/img/2024-08-28-1.webp
  alt: 카프카를 모니터링해 보자
---

## 프롤로그

> kafka + prometheus + grafana 설치되어 있는 조건에서 글을 작성했습니다.
{: .prompt-info }

kafka 를 활용한 챗봇 로그를 수집하고 분석하는 프로젝트 개발했습니다. `사용량은 꾸준하게 늘어날 예정`이고 kafka 의 실시간 상태 `모니터링이 필요` 했습니다.  
[RedHat-Kafka Exporter 설명](https://docs.redhat.com/ko/documentation/red_hat_streams_for_apache_kafka/2.1/html/using_amq_streams_on_rhel/assembly-kafka-exporter-str#con-metrics-kafka-exporter-lag-str)

현시점 가장 최신이고 꾸준하게 업데이트가 되고 있는 [danielqsj/kafka_exporter](https://github.com/danielqsj/kafka_exporter?tab=readme-ov-file) 를 채택해서 모니터링하게 되었습니다. 

## Helm Install

해당 글은 Helm Chart 를 활용했습니다. 만약 Helm Chart 를 모르시는 분이 있다면 아래 링크를 들어가서 확인해 주시면 감사합니다.  
[RedHat-helm chart 란?](https://www.redhat.com/ko/topics/devops/what-is-helm)  

### Helm 다운로드

```bash
$ git clone https://github.com/danielqsj/kafka_exporter.git
$ cd kafka_exporter/charts/kafka-exporter
$ cp values.yaml values-local.yaml
# values-local edit
```

### Helm kafka-exporter values 설정

```yaml
kafkaExporter:
  kafka:
    servers:
      - kafka.confluent.svc.cluster.local:9092
```

### Helm kafka-exporter Install 실행

```bash
$ git clone https://github.com/danielqsj/kafka_exporter.git
$ cd kafka_exporter/charts/kafka-exporter
$ helm install kafka-exporter -n monitoring . -f values.yaml
```

### Helm prometheus values 설정

```yaml
  # 3869 번째 줄
  additionalScrapeConfigs:
    - job_name: kafka-exporter
      static_configs:
        - targets:
            - "kafka-exporter.monitoring.svc.cluster.local:9308"
```

Prometheus 의 Targets 목록에 kafka-exporter 가 정상적으로 등록되었는지 확인합니다.  

![Prometheus kafka-exporter 등록 확인](/assets/img/2024-08-28-2.png)

### Grafana 대시보드 등록

대시보드의 `New -> Import` 선택합니다.   

![Grafana 신규 대시보드 등록](/assets/img/2024-08-28-3.png)  

`7589` 항목을 Load 합니다.

![7589 항목 Load](/assets/img/2024-08-28-4.png)  

등록된 `Prometheus 항목을 선택` 후 Import 합니다.

![Prometheus 항목을 선택](/assets/img/2024-08-28-5.png)

사용 중인 Kafka 모니터링이 정상적으로 동작하는 것을 확인하실 수 있습니다.

![Prometheus kafka-exporter 등록 확인](/assets/img/2024-08-28-6.png)

## 에필로그

챗봇 로그 적재 프로젝트 (WaveLens 당시 프로젝트 이름은 제가 직접 만들었습니다. 아주 잘 지었다고 생각합니다!) 에서 다음 주 일요일 특정 시간대에 트래픽이 급증하는 모습을 관찰했습니다. 
Kafka-exporter를 활용해 이를 분석하고, Kafka 브로커나 파티션 증설이 필요한지 검토했던 실무 경험이 떠오르네요. 
프로젝트는 항상 모니터링되어야 하며, 문제 발생에 대비할 수 있도록 모니터링은 필수라고 생각합니다.

[kafka-exporter + prometheus + grafana git 저장소 바로가기](https://github.com/alalstjr/kafka-exporter-note)

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Falalstjr.github.io%2Fposts%2Fkafka-exporter-%25EB%2588%2588%25EC%259C%25BC%25EB%25A1%259C-%25EB%25AA%25A8%25EB%258B%2588%25ED%2584%25B0%25EB%25A7%2581%25ED%2595%25B4-%25EB%25B3%25B4%25EC%259E%2590%2F&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)
