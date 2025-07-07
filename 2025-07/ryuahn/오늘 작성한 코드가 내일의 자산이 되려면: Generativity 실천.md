# 🌱 오늘 작성한 코드가 내일의 자산이 되려면: Generativity 실천

개인적으로 받은 교육에서 인상 깊었던 개념이 있습니다. Jessica Kerr의 "Generativity" 개념이었습니다.

> "미래의 수혜자는 다른 사람만이 아닙니다. 3개월 후의 나 자신일 수도 있습니다."

Generativity는 지금 하는 작업이 미래의 팀과 조직에 긍정적 영향을 미치는지로 생산성을 판단하는 개념입니다. <br>

교육 후 원글을 찾아 읽으며, 우리 팀에 어떻게 기여할 수 있을지 고민해보았습니다. <br>
이 글은 그 고민의 결과물입니다.

 <br>

### 📋 핵심 요약
- Generativity: 현재 작업이 미래 팀과 조직에 미치는 긍정적 영향
- 측정 기준: 개인 성과보다 팀 전체의 지속 가능한 성장에 초점
- 실천 방법: 문서화, 지식 공유, 멘토링을 통한 팀 역량 강화
  
 <br>
 
## Generativity란 무엇인가?
> "현재 우리의 행동이 미래의 시스템과 사람에게 미치는 영향" - Jessica Kerr <br>
단순히 결과물의 양이 아닌, 미래에 남기는 가치의 지속 가능성이 Generativity의 핵심입니다. <br>
개인의 성과를 넘어 조직 전체가 지속적으로 성장할 수 있는 환경을 만드는 것이 Generativity가 지향하는 방향입니다.


<br>


## 개발 생산성에 대한 새로운 기준
생산성 측정에는 여러 접근 방법이 있습니다. 개발 속도와 작업량 외에도 다음과 같은 요소들을 고려하는 관점이 있습니다.

- **협업 가능성**: 코드가 팀원과 공유하기 쉽도록 작성되어야 합니다
- **유지보수성**: 구조가 명확하여 변경이 용이해야 합니다
- **자동화 수준**: 반복적인 작업은 자동화되어 팀 전체의 효율성을 높여야 합니다
- **지식 이전 가능성**: 개인의 지식이 문서, 코드 리뷰 등을 통해 조직 전체에 공유되어야 합니다

 ### 지속 가능한 속도(Sustainable Pace)
Generativity 실천을 위해서는 '지속 가능한 속도'를 유지하는 것이 중요합니다. <br>
무리한 속도로 단기 목표만 달성하는 것이 아니라, 장기적으로 건강한 개발 환경을 유지하는 속도를 설정해야 합니다.

<br>
<br>
 
## Generativity가 조직에 주는 이점 - 회복탄력성
Generativity를 실천하면 조직의 회복탄력성(Resilience)이 향상됩니다. <br>
회복탄력성이란 문제가 발생해도 조직이 정상적으로 작동하는 능력을 의미합니다. <br>

- 특정 인력이 부재해도 업무가 지속됩니다
- 신규 구성원이 빠르게 업무를 이해하고 온보딩합니다
- 유지보수 비용이 줄어들어, 더 많은 시간을 신규 기능 개발에 투자할 수 있습니다

#### 📈 회복탄력성 향상 사례

```java
// Before: 개인 지식에 의존하는 코드
public class SettlementService {
    // 복잡한 로직이 주석 없이 작성됨
    public void processSettlement(String merchantId, LocalDate targetDate) {
        ...
    }
}

// After: 팀이 이해할 수 있는 코드
public class SettlementService {
    
    /**
     * 일일 정산 처리
     * 1. 정산 대상 거래 조회 (승인완료, 취소)
     * 2. 수수료 계산 (가맹점별 차등 적용)
     * 3. 정산 금액 산출 (매출 - 수수료 - 세금)
     * 4. 정산 데이터 생성 및 저장
     */
    public SettlementResult processDailySettlement(String merchantId, LocalDate targetDate) {
        // 1단계: 정산 대상 거래 조회
        List<Transaction> transactions = getSettlementTargetTransactions(merchantId, targetDate);
        
        // 2단계: 수수료 계산
        BigDecimal feeAmount = calculateFeeAmount(transactions, merchantId);
        
        // 3단계: 정산 금액 산출
        BigDecimal settlementAmount = calculateSettlementAmount(transactions, feeAmount);
        
        // 4단계: 정산 데이터 생성 및 저장
        SettlementData settlementData = createAndSaveSettlementData(merchantId, targetDate, 
                                                                    transactions, feeAmount, settlementAmount);
        
        return SettlementResult.success(settlementData);
    }
    
    // 각 단계별 메서드 분리로 가독성 향상
    private List<Transaction> getSettlementTargetTransactions(String merchantId, LocalDate date) { ... }
    private BigDecimal calculateFeeAmount(List<Transaction> transactions, String merchantId) { ... }
    private BigDecimal calculateSettlementAmount(List<Transaction> transactions, BigDecimal feeAmount) { ... }
    private SettlementData createAndSaveSettlementData(String merchantId, LocalDate date, 
                                                       List<Transaction> transactions, 
                                                       BigDecimal feeAmount, BigDecimal settlementAmount) { ... }
}
```

단순히 예외 처리나 시스템 이중화가 아니라, <br>
팀이 스스로 학습하며 변화와 장애에 유연하게 대응하는 환경을 만드는 것이 핵심입니다.

<br>
<br>

---
**🤔 잠깐!**

![심각한곰돌이푸](./심각한곰돌이푸.png)

여기까지 읽으면서 "이거 회사만 좋은 거 아니야?" 라고 생각하셨나요?  

사실 Generativity의 진짜 매력은 **개인의 성장**에 있습니다.

---

<br>


## 🚀  Generativity가 개인에게 주는 이점
Generativity는 조직뿐 아니라 개인의 업무 효율성과 성장에도 긍정적 영향을 줍니다.

- 문서화와 지식 공유로 반복 업무가 감소하여 개인 업무 효율이 높아집니다
- 코드 리뷰와 멘토링 등 협업 과정에서 개인 역량이 발전합니다
- 내가 작성한 문서와 코드가 팀에서 신뢰와 협력의 기반이 됩니다

<br>
<br>

## 우리는 Generativity를 실천하고 있는가?
Generativity 실천을 위해 다음과 같은 행동들을 고려해볼 수 있습니다.<br>

 <br>
 <br>

> - [ ]  **의도 문서화**: 작업의 의도를 명확한 문서로 남깁니다
> ```java
> // Before: List<User> users = getUsers();
> // After: List<User> activeUsers = getActiveUsers(); // 활성 상태인 사용자만 조회
> ```

> - [ ]  **맥락 공유**: 코드 리뷰 시 맥락과 배경을 자세히 공유합니다

```java
// 왜 이렇게 구현했는지 배경 설명
private void validateMerchantStatus(String merchantId) {
    // 2024.01.15 - 정산 오류 이슈로 인해 추가
    // 휴면 상태 가맹점의 경우 정산 대상에서 제외 필요
    // 관련 이슈: JIRA-1234
    if (merchant.getStatus() == MerchantStatus.DORMANT) {
        throw new SettlementException("휴면 가맹점은 정산 불가");
    }
}
```

> - [ ]  **지식 나눔**: 페어 프로그래밍, 멘토링을 통해 적극적으로 지식을 나눕니다

```java
// 페어 프로그래밍 중: 신입 개발자와 함께 코딩 예시

// 멘토: "결제 금액 계산할 때 왜 BigDecimal을 쓸까요?"

	@Test
	void double_소수점_정확도_테스트() {
		// double 사용 시 문제점
		double amount = 100.1 - 100.0;

		System.out.println("amount: " + amount); // 0.1

		Assertions.assertEquals(0.1, amount); //실패
	}

// 멘토: "왜 실패했을까요?"
// 멘티: "어? 0.1이 아니네요..."

	@Test
	void BigDecimal_소수점_정확도_테스트() {
		// BigDecimal 사용 시
		BigDecimal preciseAmount = new BigDecimal("100.1")
				.subtract(new BigDecimal("100.0"));
		System.out.println("preciseAmount: " + preciseAmount); // 0.1
		
		Assertions.assertEquals(new BigDecimal("0.1"), preciseAmount); //성공
	}

// "PG사에서 1원 차이도 큰 문제가 될 수 있음을 프로그래밍 통해 공유"

```

> - [ ]  **학습 촉진**: 조직 내 회고와 피드백을 자주 수행하여 팀 학습을 촉진합니다

<br>
<br>
<br>

## 마무리
소프트웨어 개발에서 '미래'는 먼 훗날이 아닙니다. 지금 작성한 코드를 3개월 후에 다시 마주하는 일은 흔합니다.<br>
Generativity는 현재 내가 작성한 코드, 리뷰, 문서가 몇 달 후 팀과 조직에 자산이 되는지를 고민하는 것입니다.

미래의 수혜자는 다른 사람만이 아닙니다. 3개월 후의 나 자신일 수도 있습니다.

완벽하지 않더라도 오늘부터 PR 설명을 조금 더 자세히 쓰고,  <br>
복잡한 로직에 주석을 남기는 것부터 시작할 수 있습니다.  <br>
작은 실천이 쌓여 더 나은 개발 문화를 만듭니다.  <br>

해당 내용은 개발 부문에만 속하는 개념이 아닙니다. 어떤 비즈니스에도 어울리는 개념입니다. <br>

오늘의 행동이 미래에 지속 가능한 업무 문화를 만드는 출발점이 됩니다.
Generativity 실천을 통해 팀과 조직이 지속 가능한 성장 환경을 구축할 수 있습니다.

<br>

### 📚 참고 자료
- [Generativity - Jessitron's Blog](https://jessitron.com/2019/08/11/generativity/)
