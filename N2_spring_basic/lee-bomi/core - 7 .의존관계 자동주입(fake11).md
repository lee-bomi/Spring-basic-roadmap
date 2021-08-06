의존관계 자동주입

2. 옵션처리

   : 스프링 빈이 없을때도 동작해야할때 자동주입대상을 옵션으로 처리하는방법

   - @Autowired(required = false)
   - @Nullable
   - Optional.empty

   ![image-20210805144158714](C:\Users\이보미\AppData\Roaming\Typora\typora-user-images\128446057-dd4cd84d-9a12-4297-895e-c6d20ca88005.png)

3. 생성자주입을 선택하라!

   - 의존관계가 애플리케이션 종료할때까지 변하지 않을예정(불변)

   - 누군가 수정가능한 코드설계는 좋은설계가 아니다(불변)

   - 생성자 주입을 사용하면 필드에 'final' 키워드를 사용할 수 있다

     => 생성자에서 값이 설정되지않는 오류를 컴파일시점에서 막아준다

   - 생성자주입방식 : 프레임워크에 의존하지않고 순수한 자바언어의 특징을 살리는방법

   - 기본을 생성자주입방식, 필수값이 아닌경우에는 수정자 주입방식사용!(동시사용가능)

     

4. 롬복과 최신 트렌드

   : 의존관계주입을 자동으로 해줄때 써줘야하는 코드(생성자, 주입받은 코드)가 많다. 이걸 대신써준다

   | ![image-20210805152438346](https://user-images.githubusercontent.com/68681443/128446155-d035a65f-3ad3-4c5e-98b2-039ea8b77210.png) |
   | :----------------------------------------------------------- |
   | ![image-20210805152410305](https://user-images.githubusercontent.com/68681443/128446201-34f42657-5690-4a1d-a9db-1458f6efd79c.png) |

   👉 롬복이 자바의 annotaion processor라는 기능을 사용해서 컴파일시점에 생성자코드를 자동으로 만들어준다.

   ​		(* annotation processor : 컴파일단계에서 어노테이션 분석하고 처리하기위한 자바컴파일러에 동봉된 hook)

   👉 롬복 라이브러리의 @RequireArgsConstructor을 함께 사용하면 기능은 다 제공하면서 코드는 간결해 진다.

   

5. 조회빈이 2개이상 일때

   : ❌ NoUniqueBeanDefinitionException 오류 발생
   
   ![image-20210805161140657](https://user-images.githubusercontent.com/68681443/128446217-ce130337-99fe-414b-af3b-c52cb47d27f8.png)

6. 자동, 수동 올바른 실무 운영 기준
   - 자동 : 컴포넌트스캔
   - 수동 : Appconfig같은 클래스에 빈 등록 및 연결

