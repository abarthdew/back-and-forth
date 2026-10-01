# Spring Cloud & MSA

## 섹션 0. Microservice와 Spring Cloud의 소개

![](images/image-01.png)
![](images/image-02.png)
![](images/image-03.png)
![넷플릭스 마이크로 서비스 구성도](images/image-04.png)

- 마이크로서비스 : 복잡하게 얽힌 서비스들. 전체 서비스들을 구성하는 개별적 모듈을 독립적으로 개발, 배포, 운영할 수 있음.

![](images/image-05.png)

- 예측하지 못한 상황에서도 견딜 수 있으며, 시스템의 변동이나 불확실성 속에도 서비스 제공 가능.

![](images/image-06.png)

- 각각의 서비스를 배포하고 빌드하는 작업을 하나의 파이프라인으로 구축해 자동화. 시스템 업그레이드 또한 빠르게 진행 가능.

![](images/image-07.png)
![](images/image-08.png)
![](images/image-09.png)
![](images/image-10.png)

- Microservices로 개발됨
- CI/CD 시스템에 의해 자동 통합, 배포를 거침
- DevOps로 오류를 바로바로 고칠 수 있음
- 마이크로 서비스를 클라우드 환경에 배포하기 위해 Containers 가상화 기술을 사용

![](images/image-11.png)
![](images/image-12.png)
![](images/image-13.png)
![](images/image-14.png)
![](images/image-15.png)
![](images/image-16.png)

- 모놀리스 : 하나의 소프트에 모든 서비스를 포함시켜 개발하는 것. 서비스들이 서로 유기성을 가지고 배포됨.
- 마이크로 서비스 : 각각 서비스의 구성요소를 분리해서 개발. 분리된 서비스가 다른 서비스에 영향을 주지 않으며, 개발과 배포가 용이.

![](images/image-17.png)
![](images/image-18.png)
![](images/image-19.png)
![](images/image-20.png)
![](images/image-21.png)
![](images/image-22.png)
![](images/image-23.png)
![](images/image-24.png)

- SOA : Service Oriented Architecture

![](images/image-25.png)
![](images/image-26.png)
![](images/image-27.png)
![](images/image-28.png)
![](images/image-29.png)
![](images/image-30.png)

## 출처

- Spring Cloud로 개발하는 마이크로서비스 애플리케이션(MSA) : [https://www.inflearn.com/course/스프링-클라우드-마이크로서비스/dashboard](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4/dashboard)
