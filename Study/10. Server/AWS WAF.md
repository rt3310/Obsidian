https://techblog.woowahan.com/2699/
## WAF (Web Application Firewall)

웹 애플리케이션 방화벽(WAF)은 웹의 비정상 트래픽을 탐지하고 차단하기 위한 방화벽이다.

일반적으로 이야기하는 방화벽(FW)과는 차이점이 있는데, 단순 방화벽(FW)은 TCP/IP 레벨에 포함된 정보들을 기반으로 차단 룰을 설정한다. 하지만 **웹 방화벽은 웹 프로토콜 HTTP 정보를 바탕으로 차단 룰을 설정한다**.
WAF는 웹 해킹 공격으로부터 웹 서비스를 전문적으로 보호하기 위해 탄생한 정보 보호 시스템이라고 이해할 수 있다. 그리고 침입 탐지, 차단 시스템(IDS/IPS)와도 역할이 다르다.

FW, WAF, IDS/IPS의 역할을 정확히 구분하고 적절한 룰을 수립하여 운용하는 것이 중요하다. 따라서 이 중 하나의 솔루션을 사용하고 있다고 다른 보안 솔루션이 필요 없다는 생각은 잘못된 생각이다.

모든 서비스가 웹, 앱을 통해 고객에게 제공되도록 개발되면서 비즈니스 측면에서 서비스 제공에 많은 편의성을 가져다주었다. 하지만 정상 서비스 이용 고객뿐 아니라 공격자들에게도 편의성을 제공하게 되어 쉽게 공격의 대상이 된다.

WAF가 웹 시장에서 주목받기 전에 웹 보안은 개발자의 보안 프로그래밍 능력에 의존되었다. 이는 종종 다양한 문제점을 일으켰고, 침해사고로 이어졌다. 이를 해결하기 위해서는 많은 비용과 보안 프로그래밍에 대한 지식, 충분한 코드 리뷰, 취약점 진단, 모의 침투 테스트 등 많은 팀과 인력의 노력이 요구되기 때문에 현실적으로 완벽하기란 한계가 있다. 또한, 서비스는 정기적으로 개편되거나 변화하면서 새로운 기능이 생기거나 사라진다. 그때마다 새로운 취약점이 나타날 수 있다.
그리고 사람은 누구나 실수하기 때문에 수많은 테스트와 충분한 검토 후 서비스를 출시하더라도 보안 문제는 발생하곤 한다.

WAF는 이러한 웹 보안 문제나 실수를 보완하기 위한 방파제 역할을 하기 때문에 필요한 보안 솔루션이다.

여기까지 읽고 '아하, WAF는 모든 웹 보안 공격을 막는 솔루션이구나'라고 생각되었다면 그것 또한 오해이다. WAF가 탐지하지 못하는 공격영역이 있다. 그 중 하나는 논리적 취약점이다.

메인 페이지 주석에 다음 내용이 있다고 예를 들어보자.
```
<!- 관리자페이지는 woowa.com/admin-manager이고 계정은 manager/!@admin!@입니다. 배포시 주석 삭제 -!>
```

WAF에서 볼 때 이 주석을 보고 관리자 페이지에 접속한 접근을 공격으로 구분할 방법이 없다.

다른 예시로는 일반 사용자에게는 노출되지 않아야 할 페이지가 보이는 잘못된 권한 부여나, 불충분한 세션 관리, 서버 정보 노출 그리고 우회를 목적으로 한 파라미터 변조도 논리적 취약점에 해당한다.

이처럼 보안 솔루션은 해킹 공격의 전부가 아닌 일부를 방어해주는 것이라 올바르게 이해하고 근본적으로 서버와 애플리케이션의 보안 조치를 게을리해서는 안된다.

## 클라우드 환경에서의 WAF 구성

### VPC 프록시 구성
먼저 알아볼 구성은 **VPC내 설치형 WAF**인데, 보통 설치형 WAF라 말하는 구성은 IDC(Internet Data Center) 환경에서 Inline WAF 구성과 유사한 구성이다.
아래 그림과 같이 VPC 내에 프록시 형태로 구성할 수 있다.

![[Pasted image 20260105140412.png]]

이 구성은 네트워크 흐름 상 중간에서 모든 L7 패킷을 검증한 뒤 Origin LB로 보낸다. 이 구성을 선택하면 WAF의 리소스를 직접 관리할 수 있고 origin LB를 Private으로 숨길 수 있는 이점이 있다.

하지만 VPC 내에 구성한 WAF 리소스 자체가 하나의 주요 장애 포인트가 될 수 있다. 이해를 돕기 위해 이벤트와 같이 트래픽이 몰리는 상황을 가정해보자.

각 서비스 팀에서 **Origin 서버의 리소스는 스케일 아웃을 실시하였으나 만약, WAF 리소스는 평소와 같은 수준으로 유지한 채로 트래픽을 받게 된다면, WAF가 제대로 된 트래픽 처리를 하지 못하므로 장애가 발생**한다.
이러한 WAF에 의한 장애를 방지하려면 Origin Private ELB (WEB, WAS 리소스)와 함께 Public ELB(WAF ELB)의 리소스도 상시 모니터링을 하면서 이벤트, 공격에 대비해 확장성 정책 및 프로세스를 유연하게 적용해야 한다.
또, WAF 장애가 발생했지만, 정확한 원인 파악이 바로 되지 않는 상황이라면 우선 WAF 우회 작업을 진행할 수 있다. WAF를 우회하기 위해 간단하게 DNS를 WAF에서 직접 Origin LB를 바라보도록 변경하거나, WAF의 자체 Bypass 기능을 사용할 수 있다.

하지만 이 구성에서는 Origin LB는 Private으로 구성하였기에 Public DNS가 바로 바라보는게 불가능하므로 기능적으로 Bypass 작업을 수행해야 해서 이 부분을 고려하여 장애 대응 프로세스를 수립해야 한다.

### SECaaS 구성
![[Pasted image 20260113133702.png]]
위 그림에서 설명하는 구성은 비교적 쉽게 WAF를 도입하는 방법으로 SaaS 혹인 SECaaS라고 부리는 형태이다. 복잡한 구성 및 운용에 대한 부담 없이 간단하게 DNS 설정만으로 트래픽을 우회하여 비정상 웹 트래픽을 필터링할 수 있다.
하지만, SECaaS WAF를 제공하는 업체와의 네트워크 환경이 고려되어야 한다. 예를 들어 서비스가 서울 리전을 사용한다면 같은 서울 리전을 통해 서비스를 제공하는 SECaaS WAF 서비스를 선택하는 것이 Latency 측면에서 유리할 것이다.
단점으로는 리소스 관리를 직접 할 수 없다는 점에서 장애가 발생했을 때, 트러블 슈팅 및 대응이 느려질 수 있고, 데이터 기밀성에 대한 문제가 있다.

서비스 아키텍처를 수립할 때 처음부터 이러한 보안을 위한 아키텍처를 고려하지 않았다면 위에서 언급한 문제점들 때문에 WAF를 바로 도입하거나 운영 계획을 수립하기 어려울 수 있다.

AWS를 사용한다면 앞서 알아본 두 가지 구성보다 더 간단하게 구성할 수 있는 WAF가 있다. AWS WAF가 바로 그러하다.
AWS WAF는 WAF 도입 시 위 구성 상 문제점에 대한 고민을 해소해줄 수 있는 장점이 있다.
- Cloudfront나 ALB, API Gateway를 사용 중이면 바로 적용할 수 있다.
- 적용 대상과 동일 선상에서 동작하기에 네트워크 홉이 추가되지 않는다.
- 트래픽이 증가하는 상황에서 확장이 유연하다.
- 유지보수에 많은 노력이 들어가지 않는다.
- 장애 발생 시 원복 시나리오가 간단하다.

AWS WAF가 아키텍처 상 유리할 수는 있지만 다양한 측면에서 상용 WAF보다 부족한 점이 많다. 하지만 AWS에서도 피드백을 받아 점차 이러한 부분을 개선하려는 노력이 보인다.

## AWS WAF 서비스를 적용하기 전에

### WAF를 어디에 연동할까?
AWS WAF는 Cloudfront ALB, API Gateway에 연동할 수 있다. 이는 단순히 '선택지가 있구나'를 얘기하는 것이 아니다. 어느 곳에 연동하는 것이 보안 측면에서 더 나을지 고민해야 한다.

Cloudfront는 AWS에서 제공하는 CDN 서비스이다. 관점에 따라서 다를 수 있겠지만, 필자는 보안 측면에서 AWS WAF를 Cloudfront에 연동하는 것이 더 유리하다고 생각한다.

트래픽 급증은 이벤트와같은 예측된 상황에서도 발생하지만, DDoS와 같은 상황에서도 발생한다. **Volumetric 한 DDoS 공격에 대한 대비를 위해서라도 Cloudfront의 Global edge를 통해서 DDoS 공격이 처리된 후 VPC 내에 트래픽이 유입되는 게 더욱 유리하다**. 그리고 L7을 겨냥한 DDoS도 있어 Cloudfront에 적용된 WAF에서 적절한 임계치 룰과 L7 DoS 차단 룰이 적용되어 있을 테니, VPC에 부적절한 트래픽 유입은 적어진다.

혹은 Cloudfront와 ALB 두 곳에 전부 WAF를 연동하는 방법이 있다. Cloudfront의 WAF는 L7 DoS 및 임계치 조절, IP Blacklist 차단 규칙으로 WAF룰을 수립하고, ALB의 WAF는 웹 취약점 공격을 중점으로 룰을 구분하여 수립할 수 있다.

#### Cloudfront + WAF와 ALB + WAF를 함께 사용하는 구조
![[Pasted image 20260113134608.png]]

보안 아키텍처를 선택할 때는 한 번 선택하면 다시 바꾸기 어려울 수 있으므로 여러 가지 사항을 사전에 고려해봐야 한다. 국내 한정된 서비스라면 굳이 CDN이 필요하지 않은 서비스일 수도 있고 Cloudfront 비용도 무시할 수 없다. 그리고 위협 및 취약성, 장애 발생, 더 나아가 멀티 클라우드 가능성 등 보안만 고집할 게 아니라 여러가지 IT 전반적인 상황을 고려하여 적절한 아키텍처를 선택할 수 있어야 한다.

## Firewall Manager Service

권한 관리, 장애 및 리소스 관리, 침해사고 예방 관점에서도 멀티 계정(Account) 환경으로 분리된 서비스 구성은 여러 이점이 있다. 하지만 이러한 환경은 각 계정에서 동일한 수준의 보안 및 관리의 어려움을 가져왔다.
그래서 AWS는 Organization 기반의 서비스를 통해 이러한 문제를 해결하는 데 도움을 주는데, Firewall Manager Service(FMS)도 이 같은 목적으로 출시되었다.
AWS Config 서비스와 연계하여 누락되어 있는 Public ELB를 WAF 적용 대상에 자동 포함시킬 수 있고, 중앙에서 룰 관리 등 운영 측면에서 이점이 많은 서비스이다.

![[Pasted image 20260113135009.png]]

하지만 FMS 서비스를 완벽하게 이해하지 못한 채로 "모든 Public Subnet의 ALB를 AWS WAF의 보호대상으로 적용할거야"라는 결정을 한다면, 수많은 오탐 확인과 예외처리 업무에 시달리게 될 것이다.
위 그림의 개념 구성도로 보면 온프레미스에서 주로 사용하던 상용 WAF와 적용 방식이 크게 달라보이지 않는다. 하지만 차이는 FMS는 도메인, 서비스별 룰을 적용하기 어렵다는 점에 있다.

#### LJH-WAF(FMS)를 전사 일괄 WAF로 적용한 구성
![[Pasted image 20260113135157.png]]

네 개의 계쩡(S, W, T, M)에 네 개(쇼핑몰, 웹툰, 택시 배차, 영상)의 웹, 앱 애플리케이션 서비스를 FMS WAF에 적용했다고 가정해보자.

FMS는 이러한 상황에서 12개의 도메인에 동일한 수준의 보안룰을 적용하기에는 더할나위 없이 좋겠지만, 이들은 같은 개발언어로 개발되지 않았을 수 있고, WAF의 룰이 복잡해질수록 오탐이 발생할 가능성도 높아진다.

따라서 관리의 용이성만 보고 멀티 계정 환경에서 무조건 FMS를 통해 WAF를 적용하는 것은 좋지 못한 선택이라고 생각된다. 만약, AWS WAF의 예외처리 기능이 고도화된다면 적극적으로 활용할 가지차 있을 것 같다.

WAF를 운영할 여력이 충분하다면 최소한 서비스 별로 WAF를 구성하여 운영하여 각 서비스(쇼핑몰: S-WAF, 웹툰: W-WAF, 택시: T-WAF, 영상: M-WAF)에 맞는 보안 룰을 적용하여 운영하던지, 더 복잡한 룰 구성이 필요하다면 도메인 별로 적용하여 운영하는 것이 좋다고 생각한다.

## AWS WAF 룰 관리

### AWS WAFv2 Managed Rule의 탐지율 비교
19년 11월에 발표된 WAFv2에는 AWS Managed Rule을 제공해준다. WAFv1에서 운영하던 Custom Rule과 Marketplace Rule 조합으로 되어 있던 룰 구성보다 조금 더 다양한 룰로 구성할 수 있게 됐고, 룰에 대한 신뢰를 가지면서 운영할 수 있어졌다.

그리고 WAFv2는 WAFv1에서 10개의 룰을 사용할 수 있었던 것과 다르게 1,500 capacity를 제공하고 이 용량 안에서 적용하여 사용 가능하다. 부족하다면, AWS 측에 capacity 증가를 요청할 수 있다.

아래 표는 AWS WAFv2의 Managed Rule과 Marketplace Rule의 탐지율을 비교하기 위해 기본 용량 1,500 capacity 안에서 임의로 Managed Rule을 구성한 표이다. 이 구성은 1,475 capacity를 차지하게 된다. 실제 구성에서는 임계치 및 Custom 룰, L7 DoS 룰도 분명히 필요할 것이므로 불가피하게 몇 개의 룰을 양보하거나 capacity를 증대하여 사용하게 된다.

| 룰 이름                      | 설명                                                                                                                      | 용량  |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------- | --- |
| Core Rule Set             | 일반적으로 웹 애플리케이션에 적용 할 수있는 규칙을 포함합니다. 이는 OWASP 간행물에 설명 된 것을 포함하여 광범위한 취약성의 악용으로부터 보호합니다.                                  | 700 |
| Known bad inputs          | 유효하지 않은 것으로 알려져 있고 취약성의 악용 또는 발견과 관련된 요청 패턴을 차단할 수있는 규칙을 포함합니다. 이를 통해 악의적 인 공격자가 취약한 애플리케이션을 발견 할 위험을 줄일 수 있습니다.        | 200 |
| SQLi Rule Set             | SQL injection 룰                                                                                                         | 200 |
| Linux operating system    | LFI 공격을 포함하여 Linux에 특정한 취약성 악용과 관련된 요청 패턴을 차단하는 규칙을 포함합니다. 이렇게하면 파일 콘텐츠를 노출하거나 공격자가 액세스 할 수없는 코드를 실행하는 공격을 방지 할 수 있습니다. | 200 |
| Admin Protection Rule Set | 노출된 관리 페이지에 대한 외부 액세스를 차단할 수있는 규칙을 포함합니다.                                                                               | 100 |
| Anonymous IP list         | 여기에는 VPN, 프록시, Tor 노드 및 호스팅 공급자에서 시작된 요청이 포함될 수 있습니다. 이는 애플리케이션에서 신원을 숨기려고 할 수있는 뷰어를 필터링하려는 경우에 유용합니다.                  | 50  |
| Amazon IP reputation list | Amazon 위협 인텔리전스를 기반으로하는 규칙이 포함됩니다. 이는 봇 또는 기타 위협과 관련된 소스를 차단하려는 경우 유용합니다.                                               | 25  |

MarketPlace에서 구매 적용 가능한 Marketplace Rule과 위의 AWS Managed Rule 구성을 대상으로 두 개의 다른 웹 스캐너를 이용하여 테스트를 진행하였다.
#### A Scanner Test
|Rule|Block|Allow|Detect Rating|
|---|---|---|---|
|AWS Managed Rule|442|1,096|28%|
|Marketplace Rule|204|1,334|13%|
#### B Scanner Test
|Rule|Block|Allow|Detect Rating|
|---|---|---|---|
|AWS Managed Rule|1,743|8,936|17.9%|
|Marketplace Rule|700|9,979|5.6%|
테스트 결과 AWS Managed Rule이 Marketplace Rule보다 평균적으로 2배 이상 더 높은 탐지율을 보인다. 하지만 상용 WAF 제품보다는 확실히 부족한 탐지율이라 생각된다.
적절한 수준의 워크로드 보안을 하기 위해 AWS WAF의 Custom Rule을 통해 탐지율을 높이고 오탐율은 낮추는 노력과 함께 별도의 CWPP(Cloud Workload Protection Platform) 솔루션도 함께 사용해야 한다고 생각된다.

### AWS WARF에서 이상 트래픽을 검사하는 Custom 룰
WAF에서 Custom 룰을 수립할 경우 AND, OR, NOT 조건을 통해 논리적 규칙문을 수립할 수 있다.

| 규칙문 | 내용                                                                 |
| --- | ------------------------------------------------------------------ |
| AND | IP는 한국 IP 이다. **그리고**, URI에 XSS공격 쿼리문이 삽입되어 있는 경우 차단한다.            |
| OR  | URI에 XSS공격 쿼리문이 삽입되어 있거나 **혹은**, Body에 XSS공격 쿼리문이 삽입되어 있는 경우 차단한다. |
| NOT | URI에 ‘content’라는 문자열이 삽입되어 있지 **않은 경우**, 차단한다.                     |
이 조건문과 웹 헤더 검사문을 조합하면 유용한 룰을 수립할 수 있다. 예를 들어 Public ELB는 기본 Public 도메인 주소를 가지고 있다. 이때 공격자는 기본 Public LB Domain 주소나 LB Domain의 IP주소로 공격을 시도한다. 공격자의 목적은 'SaaS 구성'이나 'CDN + 보안서비스'와 같이 논리적 구성에서 상단에서 동작 중인 보안 솔루션을 우회하기 위한 시도이다.

이해를 돕기위해 앞서 알아본 **Cloudfront + WAF 와 ALB + WAF를 함께 사용하는 구조**에서의 Cloudfront WAF를 우회하기 위한 시도를 그림으로 설명하였다.
![[Pasted image 20260113140524.png]]

공격자는 실제 서비스 도메인 주소(example.baemin.com)가 아닌 ALB 도메인 주소(example.elb.domain.com)로 공격을 시도한다.

이 공격자의 공격을 차단하기 위한 WAF Custom 룰로 host header를 검사하여 DNS에 등록된 정식 도메인 주소를 통해 유입된 것인지 확인하고 차단할 수 있다.

물론, 이 구성에서는 ELB Security Group을 사용하여 Cloudfront가 사용하는 AWS IP를 제어하는 방법도 좋은 방법이다.

아래는 위에서 설명한 Custom 룰에 대한 WAFv2 정책(json) 예시이다.
#### Host 헤더 검사 룰
```json
{
  "Name": "Check_host_header_rule",
  "Priority": 0,
  "Action": {
    "Block"
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "Check_host_header_rule"
  },
  "Statement": {
    "NotStatement": {
      "Statement": {
        "ByteMatchStatement": {
          "FieldToMatch": {
            "SingleHeader": {
              "Name": "host"
            }
          },
          "PositionalConstraint": "EXACTLY",
          "SearchString": "www.example.com",
          "TextTransformations": [
            {
              "Type": "NONE",
              "Priority": 0
            }
          ]
        }
      }
    }
  }
}
```

WAF의 Custom 룰을 이상 트래픽 탐지에 활용할 수도 있다.
**어떤 API는 반드시 특정 페이지를 경유해서 접근되어야 하는 페이지라면, Referer 헤더를 검사하여 비정상으로 유입되는 트래픽을 확인할 수도 있다**.
Referer 헤더가 없거나 예상된 경로와 다르다면, 서비스 웹 페이지를 브라우저로 서핑 중 클릭한 트래픽이 아닌 직접 페이징 접근한 트래픽으로 간주할 수 있다.

웹 헤더는 쉽게 변조가 가능하므로 이러한 헤더 검사 룰은 100% 신뢰할 수는 없다. 하지만 WAF를 이용해 이러한 이상 트래픽을 탐지할 수 있는 인사이트를 얻는다면 공격 외 비정상 트래픽을 차단, 검사하는데 도움이 될 것이다.

아래는 Referer를 검사하는 WAFv2 정책(json) 예시이다.
#### referer 헤더 검사 룰
```json
{
  "Name": "referer_check",
  "Priority": 0,
  "Action": {
    "Block"
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "referer_check"
  },
  "Statement": {
    "AndStatement": {
      "Statements": [
        {
          "ByteMatchStatement": {
            "FieldToMatch": {
              "SingleHeader": {
                "Name": "referer"
              }
            },
            "PositionalConstraint": "CONTAINS",
            "SearchString": "www.example.com/goods/detail",
            "TextTransformations": [
              {
                "Type": "NONE",
                "Priority": 0
              }
            ]
          }
        },
        {
          "ByteMatchStatement": {
            "FieldToMatch": {
              "UriPath"
            },
            "PositionalConstraint": "CONTAINS",
            "SearchString": "payment",
            "TextTransformations": [
              {
                "Type": "NONE",
                "Priority": 0
              }
            ]
          }
        }
      ]
    }
  }
}
```

## AWS WAF 로깅과 탐지

### AWS WAF의 이벤트 로그를 로깅하는 법
AWS WAF를 100% 활용하기 위해서는 로깅과 로그 분석은 필수이다.

WAF 로그 분석은 WAF를 통해 해결할 수 있는 보안 위협 공격, 취약점 공격을 포함하여 여러 웹, 앱 서비스를 운영하며 생길 수 있는 문제들을 효과적으로 해결할 수 있다.

콘솔 상 자체 로그를 일부 남기기도 하고, Cloudwatch로 그래프를 제공하지만 적절한 분석을 하기에는 부족하다.

현재 AWS WAF에서는 Kinesis Firehose 서비스를 이용한 로깅만 지원하고 있으며, Amazon ES와 S3에 쌓을 수 있다.
별도 SIEM(Security Information and Event Management) 솔루션을 구축한 상태가 아니라면 Amazon ES 서비스나 S3에 쌓은 로그를 AWS Athena 서비스를 이용해 분석하는 것도 좋지만, 장기적으로 여러 보안서비스 및 솔루션들을 활용할 계획이 있다면 전문 SIEM 솔루션을 활용하여 분석하는 것이 좋다고 생각한다.ㅔ

이 글에서는 Kinesis Firehose를 이용하여 S3에 적재하는 방법을 설명한다.
클라우드 환경에서 로그 분석과 관리의 어려움을 해소하기 위해 적절한 중앙 로깅 아키텍처 설계가 선행되어야 한다.
중앙 로깅 아키텍처 설계를 풀이하자면 각 계정의 서비스나 솔루션에서 발생하는 로그를 한 곳에 집중시켜 수집하는 것인데, 이 구성으로 로그 보관과 접근 제어, 주요 로그 접근에 대한 감사추적 등 관리의 효율성을 얻을 수 있다.

예를 들어, A Account의 A-WAF와 B Account의 B-WAF, C Account의 C-WAF가 있으면, 각 A, B, C의 S3 서비스에 수집하는 것이 아니라, 중앙 수집할 별도 S(Security) 계정의 S3에 'Security-WAF-Bucket'을 생성하여 수집한다.
#### WAF 로그 중앙 수집 아키텍처
![[Pasted image 20260113163610.png]]

위 그림과 같이 구성하기 위해서 Kinesis Firehose를 생성할 때, S-Account의 S3 Bucket을 직접 바라보도록 설정해야 한다.
웹 콘솔에서는 같은 Organization이라도 다른 계정의 Bucket을 직접 바라보게 설정할 수 없으므로 AWS CLI로 강제 설정하여 생성해준다.
```bash
 aws firehose create-delivery-stream --delivery-stream-name aws-waf-logs-security-A-to-S-service-firehose --delivery-stream-type DirectPut --s3-destination-configuration file://./firehoseConfig.json
```

WAF가 사용할 수 있는 Firehose는 공통으로 "aws-waf-logs"라는 이름으로 시작해야 한다.
FirehoseConfig.json은 아래와 같이 생성한다.
RoleARN에는 accountNumber와 Firehose가 사용할 Role을 지정해주고, BucketARN에 대상 버킷 정보를 정확하게 적어준다. 로그를 쌓는 대상을 구분하기 쉽게 Prefix 부분도 신경 써서 작성해주고 그 외 설정도 필요한 옵션을 수정하여 생성한다.
```json
{
  "RoleARN": "arn:aws:iam::accountNumber:role/firehose_delivery_role",
  "BucketARN": "arn:aws:s3:::Security-WAF-Bucket",
  "Prefix": "wafv2/accountNumber/servicename/",
  "ErrorOutputPrefix": "string",
  "BufferingHints": {
    "SizeInMBs": 3,
    "IntervalInSeconds": 60
  },
  "CompressionFormat": "GZIP",
  "EncryptionConfiguration": {
    "NoEncryptionConfig": "NoEncryption"
  },
  "CloudWatchLoggingOptions": {
    "Enabled": true,
    "LogGroupName": "waf-firehose-testing",
    "LogStreamName": "logStream"
  }
}
```

예제로 만든 S-Account의 Bucket인 Security-WAF-Bucket에도 Policy를 설정해주어야 한다.
아래는 WAF 로그를 운반할 Firehose가 사용하는 firehose_delivery_role이 S3에 로그를 적재할 수 있도록 S3 정책을 열어주는 예시이다.
```json
 {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowRole",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::accountNumber:role/firehose_delivery_role"
            },
            "Action": [
                "s3:Put*",
                "s3:List*",
                "s3:Get*"
            ],
            "Resource": [
                "arn:aws:s3:::Security-WAF-Bucket/*"
            ]
        }
    ]
}
```

로그를 적재할 때, 한 가지 더 고민해야 할 부분이 있다.
AWS WAF를 블랙리스트 룰 기반으로 운영한다고 할 때, 전체 로그는 (Default Action) Allow, Count, Block 로그로 나눌 수 있다.
AWS WAF는 로깅 시 이 세 가지 형태의 로그를 Kinesis Firehose에 모두 실어 보낸다. AWS WAF, Kinesis Firehose 자체 기능으로는 어떤 로그만 골라 수집하겠다는 기능은 없다.

Allow 로그는 분명 공격 미탐(False Negative), 숨겨진 공격, 침해사고 조사, 이상(Anomaly)로그 분석 시 유용하게 쓰일 수 있는 로그이다. 하지만 경우에 따라서 굳이 WAF Allow 로그는 분석하지 않아도 될 로그로 볼 수 있다. 예를 들어 서버 Accesslog를 별도로 수집하고 있거나, ELB Accesslog를 수집하고 있는 경우에 그렇다.
혹은 SaaS기반 SIEM이나 ES 등이 로그의 총량에 따라 비용이 책정되는 상황이라면, 라이선스나 DB를 늘리는 선택보다는 전체 WAF 로그의 80~90%를 차지하게 될 Allow 로그를 버리는 선택을 할 수도 있다.

Allow 로그를 버리기로 하였다면, 두 가지 방법으로 로그를 버릴 수 있는데, 첫 번째는 Firehose와 lambda 트리거를 이용한 방법이다.
#### lambda를 이용하여 Count, Block 로그를 정제하는 구성
![[Pasted image 20260113164327.png]]
위 그림에서 각 Firehose에 lambda를 트리거 시켜서 lambda가 Allow 로그를 거르는 작업을 하고, Count, Block 로그만 정제하여 반환하여 S3 Security-WAF-Bucket에 쌓도록 한다.

물론, 이 그림에서 설명하는 구성은 단순 예제이고 이와 다르게 lambda를 구현하는 방식에 따라 All-WAF-Bucket, Count-WAF-Bucket, Block-WAF-Bucket을 구분하여 적재하거나 하나의 Bucket 내에 폴더, 경로(Block/, Count/, All/)를 생성하여 쌓을 수도 있다.

#### logstash를 이용하여 Count, Block 로그를 정제하는 구성
![[Pasted image 20260113164513.png]]
다른 방법은 WAF가 바라보는 S3까지는 WAF의 전체 로그(Allow, Count, Block)를 적재하고 다시 그 버킷에서 로그를 Logstash로 당겨서 Count, Block 로그를 구분하여 별도의 새 버킷에 저장할 수 있다.
이 또한, lambda 구현과 같게 Logstash 구현 방법에 따라 여러 구성이 생성될 수 있을 것 같다.

lambda를 사용하는 것과 Logstash를 사용하는 것의 로깅 Latency, 효율성, 코드 유지보수 및 비용 등을 잘 고려하여 결정하자.

## AWS WAF의 탐지 데이터

WAF의 보안 룰에 의한 공격 차단은 공격인 경우를 정확히 탐지(True Positive)하는 게 목적이지만, 기대와 다르게 오탐(False Positive)이 발생할 수 있다.

정탐과 오탐이 발생할 경우 그 이유를 분석하고 확인하는 것은 보안 분석가의 업무 중 하나이다. 이때 활용할 수 있는 정보가 **Matched on Data**이다.

하지만 AWS WAF는 HTTP의 다양한 웹 메소드(GET, POST, PUT, DELETE, ...) 중에서 Body 값을 가지고 있는 메소드로 유입된 트래픽의 Body 값은 WAF가 검사는 하지만 로그로 남기지 않는다.
심지어 Body 값에서 매칭된 패턴에 의해 차단이 되었다고 해도, 그 이유(Matched on data)를 기록하지 않는다. 이 경우 공격 벡터가 오탐 문자열이 Body 값에 포함되어 유입될 경우 정/오탐 여부를 판단하기가 어려워진다. 불행 중 다행으로 SQLi와 XSS Rule에 의해 탐지된 데이터는 Body 값에 있더라도 남겨준다.
모든 공격에 대해 Body 값의 탐지 데이터를 기록하지 않는 것은 AWS WAF에서 가장 아쉬운 부분이다.

![[Pasted image 20260113164933.png]]
위 그림처럼 SQLi와 XSS에 의해 탐지된 데이터는 terminatingRuleMatchDetails라는 필드에 의해 기록된다. 빠른 시일 내에 모든 공격에 대해 Body 값 내 탐지 데이터가 남겨지기를 기대한다.

## 정리

AWS WAF는 기능 면에서 아직 미성숙한 점이 있어 상용 WAF의 완전한 대체가 될 수는 없다고 생각한다.
룰 예외처리나 모니터링, 탐지 값 제공에 대한 기능적 아쉬움이 있고, 성숙한 WAF는 전문적인 봇 트래픽에 대한 인사이트도 제공해주지만 AWS WAF는 아직 1차원적인 속성을 통한 봇 탐지밖에 구현하지 못한다.

그리고 상용 솔루션보다 운영이 간단하고 쉬어 보여도 상용 WAF와 다른 부분이 많아 제대로 학습하지 않은 상태에서 운영한다면 의도하지 않은 차단, 오탐으로 인한 장애나 가용성 문제의 대상이 될 수 있다.

하지만 AWS WAF는 클라우드 환경에서 상용 WAF 솔루션을 도입하는 것보다 저렴하고, 안전하고, 빠르고 쉽게 적용할 수 있는 웹 애플리케이션 보안 서비스이다.
상용 솔루션을 도입하기 전 임시로 사용하거나 잘 운영하여 적절한 솔루션을 찾기 전까지 메인 웹 방화벽으로 사용하기에도 기본 이상의 가치를 제공해주는 보안 서비스라고 생각된다.