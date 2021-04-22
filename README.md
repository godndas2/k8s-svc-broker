# Overview

다양한 클라우드 서비스를 연계할 일이 생겼을 때 컨테이너 기반 애플리케이션에 클라우드 서비스를 연결하려면 번거로운 설정작업들이 필요합니다. 그래서 등장한 것이 [쿠버네티스 서비스 카탈로그](https://github.com/kubernetes-sigs/service-catalog)와 [서비스 브로커](https://www.openservicebrokerapi.org/)입니다.

# Open Service Broker

![https://www.cloudfoundry.org/wp-content/uploads/osbapi-graphic-1024x461.jpg](https://www.cloudfoundry.org/wp-content/uploads/osbapi-graphic-1024x461.jpg)

k8s는 ServiceBroker(SB)가 사용 가능한 서비스 목록을 **카탈로그**에 등록할 수 있도록 해주는 [Service Catalog Projec](https://github.com/kubernetes-sigs/service-catalog)t를 가지고 있습니다. 사용 권한이 있는 User는 Service Catalog(SC)를 요청할 수 있습니다. k8s SC를 Install 하는 방법은 여러가지가 있는데 [Helm](https://github.com/kubernetes-sigs/service-catalog/blob/master/docs/install.md)에서도 제공이 됩니다. 또, SC에는 [svcat](https://svc-cat.io/docs/cli/)이라는 Process를 쉽게 생성하는 CLI도 제공됩니다.

# Service Catalog

서비스 카탈로그에는 서비스 브로커가 제공 할 수있는 모든 사용 가능한 서비스를 설명하는 메타 데이터가 포함되어 있습니다.

User는 Catalog를 탐색할 수 있고 Service 실행에 필요한 Pod, Service, Configmap 등을 다루지 않고도 Catalog에 나열된 Service Instance를 provisioning 할 수 있습니다.

## Service Catalog의 4가지 API

### **ClusterServiceBroker**

- 서비스를 프로비저닝할 수 있는 외부 시스템을 기술한다.

### **ClusterServiceClass**

- 프로비저닝할 수 있는 서비스 유형을 기술한다.

### **ServiceInstance**

- 프로비저닝된 서비스의 한 인스턴스이다.

### **ServiceBinding**

- 클라이언트(파드) 세트와 ServiceInstance 간의 바인딩을 나타낸다.

![https://user-images.githubusercontent.com/6982740/94819184-ecf41c00-0439-11eb-80cb-426d6f9c2885.png](https://user-images.githubusercontent.com/6982740/94819184-ecf41c00-0439-11eb-80cb-426d6f9c2885.png)

![https://user-images.githubusercontent.com/6982740/94819184-ecf41c00-0439-11eb-80cb-426d6f9c2885.png](https://user-images.githubusercontent.com/6982740/94819184-ecf41c00-0439-11eb-80cb-426d6f9c2885.png)

- 클러스터 관리자는 클러스터에서 서비스를 제공하고자 하는 각 SB에 ClusterServiceBroker 리소스를 생성한다.
- 사용자는 Service를 provisioning 해야 하는 경우 ServiceInstance Resource를 생성한 다음 ServiceBinding을 사용해 해당 ServiceInstance를 Pod에 binding한다.
- 해당 Pod에는 provisioning 된 ServiceInstance에 연결하는 데 필요한 모든 자격 증명과 기타 데이터가 포함된 Secret이 주입 된다.

## Catalog Service Component

- API Server
- etcd
- Controller Manager
- Service Catalog Resource를 API 서버에 생성하고, etcd에 저장한다.
- 요청된 서비스는 Service Catalog API에서 SB Resource를 생성해 등록된 External SB에게 provisioning을 위임한다.

# Service Broker

모든 broker는 OpenServiceBroker API를 구현해야 한다.

[https://github.com/openservicebrokerapi/servicebroker](https://github.com/openservicebrokerapi/servicebroker)

### **OpenServiceBroker API 소개**

**서비스 목록 검색 GET** 

/v2/catalog - returns information about service offerings and service plans provided by service broker

**서비스 인스턴스 프로비저닝 PUT** 

/v2/service_instances/:instance_id - creates a new service instance with the specified unique ID, service offering and service plan

**서비스 인스턴스 업데이트** **PATCH** 

/v2/service_instances/:instance_id - updates an existing service instance to change service plan 

**서비스 인스턴스 바인딩 PUT** 

/v2/service_instances/:instance_id/service_bindings/:binding_id - bind a service instance to an application by specified unique binding ID

**서비스 인스턴스 바인딩 해제 DELETE** /v2/service_instances/:instance_id/service_bindings/:binding_id - remove a service application binding for a specified service instance

**서비스 인스턴스 디프로비저닝 DELETE** /v2/service_instances/:instance_id - remove a service instance

# Example

- 다른 유형의 Api service를 provisioning 할 수 있는 가상의 broker이다.
- **GET** /v2/catalog (서비스 목록 검색) → 각 service에 대한 ClusterServiceClass Resource를 생성한다.

## Service Catalog에 Broker 등록하기

```yaml
apiVersion: servicecatalog.k8s.io/v1alpha1
kind: Broker
metadata:
  name: huhyun-broker   # broker name
spec:
  url: http://broker-url.com # Service Catalog 가 broker에 연결할 수 있는 url
```

# URL Properties

This specification defines the following properties that might appear within URLs:

- service_id
- plan_id
- instance_id
- binding_id
- operation
