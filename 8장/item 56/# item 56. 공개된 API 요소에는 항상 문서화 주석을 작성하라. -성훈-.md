# item 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라

API를 쓸모 있게 하려면 잘 작성된 문서도 곁들어야 한다. (API 문서화)

전통적으로 API 문서는 사람이 직접 작성하므로 코드가 변경되면 매번 함께 수정해줘야 하는데,
자바에서는 `자바독(javadoc)`이라는 유틸리티가 이 귀찮은 작업을 도와준다.

자바독은 소스코드 파일에서 `문서화 주석(doc comment; 자바독 주석)` 이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다.

## 자바독 (Javadoc)

Java 소스에 문서화를 하는 방법으로 클래스나 메소드에 주석으로 기술한 내용을 javadoc 명령어나 또는 이를 이용한 빌드 툴(maven의 pharse) 을 사용하여 문서화 할 수 있다.

문서화할 주석을 작성할 경우 `/**` 두 개로 시작하고 `*/`로 끝나야 한다.

특별한 키워드는 `@keyword` 형식으로 작성한다.

![](https://velog.velcdn.com/images/shkim1199/post/1e78c17a-3c88-4386-b974-f80724d2da29/image.png)

자바독의 형식은 보통 위처럼 `소스 파일에 대한 설명`을 위해서 쓰기도 하고 아니면 javadoc으로 export 하여 html 로 직접 확인할 수 있는 방법으로 쓰인다.

| 어노테이션       | 의미                                                  |
| ---------------- | ----------------------------------------------------- |
| **@author**      | 코드 소스 작성자                                      |
| **@deprecated**  | Content Cell                                          |
| **@exception**   | 예외처리할 수 있는 것들을 정의, 알파벳 순             |
| **@param**       | 매개변수 메서드, 생성자 설명                          |
| **@return**      | 리턴값 설명                                           |
| **@see**         | 파일이 참조하는 다른 클래스와 메서드 등               |
| **@serial**      | Serializeable 인터페이스에 사용                       |
| **@serialData**  | writeObject writeExternal 메소드로 작성된 데이터 설명 |
| **@serialField** | serialPersistnetFields 모든 배열에 사용됨             |
| **@since**       | 해당 클래스가 추가된 버전                             |
| **@throw**       | @exception처럼 예외처리하는 것들을 정의               |
| **@version**     | 구현체, 패키지 버전 명시                              |

![](https://velog.velcdn.com/images/shkim1199/post/ccc82381-de3c-4886-b423-56b03938b254/image.png)

> 이렇게 필요한 부분만 작성하면 된다.

### 사용법

```bash
// 명령어 사용

$ javadoc -d docs {file_name}
```

```bash
// 한글을 사용하면 UTF-8 인코딩 필요

$ javadoc -d docs *.java -encoding UTF-8 -charset UTF-8 -docencoding UTF-8
```



이런식으로 생김

### 예제

```java
package com.example.basiccrud.doc;

public class GameCharacter{
    private int level;
    private String name;
    private String weapon;
    private Job job;
    /**
     * 게임 캐릭터를 생성합니다.
     * <p>기본 무기는 목검, 기본 직업은 Beginner입니다.
     * @param name 캐릭터의 이름; 길이는 3자 이상 10자 이하이어야 합니다.
     * @throws IllegalArgumentException 캐릭터의 name 길이가 정해진 범위를 벗어나면, 즉 ({@code name < 3 || name > 10}) 이면 발생합니다.
     */
    public GameCharacter(String name) {
        this.level = 1;
        if (name.length() < 3 || name.length() > 10) throw new IllegalArgumentException("캐릭터의 이름은 3자 이상 10자 이하입니다.");
        this.name = name;
        this.weapon = "목검";
        this.job = Job.Beginner;
    }

    /**
     * 캐릭터의 레벨을 반환합니다.
     * @return 캐릭터의 레벨
     */
    public int getLevel(){
        return this.level;
    }

    /**
     * 캐릭터의 직업을 변경합니다.
     * @param job 캐릭터의 변경할 직업
     * @throws IllegalArgumentException 캐릭터의 레벨이 10이 넘지 않았다면 발생합니다.
     */
    public void setJob(Job job){
        if (this.level < 10) throw new IllegalArgumentException("캐릭터의 레벨이 10을 넘지 않습니다.");
        this.job = job;
    }

    /**
     * 캐릭터의 무기를 변경해주는 메서드입니다.
     * @param weapon 캐릭터가 착용할 무기
     * @param weaponLevel 무기의 레벨
     * @throws IllegalArgumentException 캐릭터의 레벨보다 무기의 레벨이 높으면 발생합니다.
     */
    public void setWeapon(String weapon, int weaponLevel){
        if (weaponLevel > this.level) throw new IllegalArgumentException("캐릭터의 레벨보다 무기의 레벨이 높습니다.");
        this.weapon = weapon;
    }

    /**
     * 캐릭터의 레벨을 올려주는 메서드입니다.
     */
    public void levelUp(){
        this.level++;
    }

    /**
     * 캐릭터의 status값을 보여주는 메서드입니다.
     * @return 직업, 레벨, 이름, 무기를 반환합니다.
     */
    public String getCharacterStatus() {
        return "GameCharactor [job=" + job + ", level=" + level + ", name=" + name + ", weapon=" + weapon + "]";
    }

    /**
     * 캐릭터의 직업을 나타냅니다.
     */
    public enum Job{
        /** 초보자 */
        Beginner,
        /** 전사 */
        Warrior,
        /** 마법사 */
        Wizard,
        /** 궁수 */
        Archer,
        /** 도적 */
        Thief
    }
}

```

![](https://velog.velcdn.com/images/shkim1199/post/8261750f-d732-4164-9f8c-416394464ea6/image.png)



![](https://velog.velcdn.com/images/shkim1199/post/43877b0f-8ad5-4726-93a8-b98824ce32c3/image.png)

![](https://velog.velcdn.com/images/shkim1199/post/ffcfd7ad-13eb-46d4-b7d3-fe0ff4e19a80/image.png)

 [index.html](C:\Users\shkim\OneDrive\바탕 화면\suverhoon\hoonji-main\src\main\java\com\example\basiccrud\doc\docs\index.html) 



### 주의해야 할 점

#### 1. api 를 올바르게 문서화하려면 공개된 모든 클래스 ,인터페이스 ,메서드, 필드 선언에 문서화 주석을 달아야 한다.

* 주석을 달지 않으면 그저 공개된 api 요서들의 '선언' 만을 나열한다.
* 공개 클래스는 기본 생성자에 주석을 달 수 있는 방법이 없으니 절대 기본 생성자를 사용해서는 안된다.

#### 2. 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.

* how가 아닌 what을 기술 (상속용으로 설계된 api가 아닌 이상)
* 메서드를 성공적으로 호출하기 위한 전제조건을 나열
* 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건을 나열할 것
* 부작용도 문서화할 것(백그라운드 스레드를 시작하는 메서드)



---

# 핵심

> * 문서화 주석은 api를 문서화하는 가장 훌륭하고 효과적인 방법이다.
>
> * 공개 api라면 빠짐없이 설명을 달아야 한다.
>
> * 표준 규약을 일관되게 지키자
>
> * 문서화 주석에 임의의 HTML 태그를 사용할 수 있음을 기억하자
>
>   
