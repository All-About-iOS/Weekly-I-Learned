네트워킹을 하기 위해 서버가 보내주는 JSON 데이터의 타입과 속성의 이름을 반드시 일치시켜야 합니다.
그럼 클라이언트 개발자는 API 명세를 보고 꼼꼼히 입력해야할텐데요,,, 만약 속성이 어어어어어ㅓㅓ엄청 많다면..?
휴면 에러가 발생하기 딱 좋을 것 같은데요^ㅡ^

[https://app.quicktype.io](https://app.quicktype.io)

그래서 이렇게 JSON 모델을 손쉽게 만들어주는 툴이 존재합니다!

물론 항상 원하는대로 나오는 것은 아니라 약간의 수정이 필요하기도 하지만 이정도의 귀찮음은 감수 가능하죠^,^
그런데, 서버가 보내주는 Response 모델에 모든 속성을 사용하지 않는다면? 불필요한 속성까지 모두 작성해주어야 할까요?
우선 저는 서버의 Response 모델과 조금이라도 다르면 에러가 난다고 생각해서 불필요한 속성도 다 기입해주었습니다.

Open API의 경우 제 입맛대로 바꾸긴 당연히 어렵고
개인 프로젝트를 진행할 때 서버 개발자분들에게 '이 API에 안쓰이는 정보가 너무 많아요, 수정 부탁드립니다.'라는 요청을 꽤 많이 드렸는데요ㅎㅎ,,~

알고보니 내가 필요한 속성만 명시해주어도 문제 없이 네트워킹이 진행된다는 사실!!!

예를 들어, 네이버의 검색 결과를 받아오는 Open API가 있습니다.

[https://developers.naver.com/docs/serviceapi/search/shopping/shopping.md#](https://developers.naver.com/docs/serviceapi/search/shopping/shopping.md#)

해당 API의 Response 모델을 살펴보면, 

``` swift
struct SearchResponse: Codable {
    let lastBuildDate: String
    let total, start, display: Int
    let items: [Product]
}

struct Product: Codable {
    let title: String
    let link: String
    let image: String
    let lprice, hprice, mallName, productID: String
    let productType: String
    let brand: Brand
    let maker: String

    enum CodingKeys: String, CodingKey {
        case title, link, image, lprice, hprice, mallName
        case productID = "productId"
        case productType, brand, maker
    }
}
```

이렇게 엄청 많은 정보가 담겨서 오는데요!
하지만 저는 몇개만 골라서 사용하고 싶어서


``` swift
struct SearchResponse: Codable {
    let total, start, display: Int
    var items: [Product]
}

struct Product: Codable { }
```


이렇게 정리해주었어요.
여기서 궁금한 점은, 같은 API를 호출할 때 Codable한 모델의 속성 개수 차이가 성능에 영향을 줄까? 였습니다.


그래서 간단하게 확인해보니!!

![image](https://github.com/All-About-iOS/Weekly-I-Learned/assets/52594310/41252669-3038-4c24-b728-ebddd36e02d6)
![image](https://github.com/All-About-iOS/Weekly-I-Learned/assets/52594310/bcb9af53-3978-4e7c-8c31-1454e85505ea)


유의미한 차이는 발견하지 못했습니다^^,,
어떻게 보면 당연한 결과 같기도 하네요!

그럼 아예 서버가 보내주는 Reponse 모델 자체를 간소화시킨다면?

Open API로는 테스트가 불가능하기 때문에 저의 개인적인 의견을 말씀드리자면ㅎ..
단순히 속성의 개수가 성능에 영향을 미치기보다는
여러 속성을 전달하기 위해 여러 데이터베이스를 join하고 필터링 하는 과정이 얼마나 복잡한지, 일부 속성을 제외했을 때 연산 과정이 얼마나 간소화되는지에 따라 성능이 달라질 것 같습니다..!

여러분들은 어떻게 생각하시나요??
