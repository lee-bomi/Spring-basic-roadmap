# Spring MVC 1

### 7. 웹 페이지 만들기

1. 요구사항 분석

   - 2-3번 : 핵심 도메인 로직(서비스는 생략함)

2. 도메인vo만들기

3. Repository만들기

4. 2,3번에 대한 테스트진행

5. HTML

6. ModelAttribute

   - 파라미터로 primitive타입이 오면 @RequestParam이 적용되고,

   - 객체타입이 오면 @ModelAttribute가 적용된다(둘다 생략가능!)

     ![image](https://user-images.githubusercontent.com/68681443/132271807-d2029107-18bc-44fd-ade6-102ae4a80990.png)

     ```java
     //    @PostMapping("/add")
         public String addItemV1(
                 @RequestParam String itemName,
                 @RequestParam int price,
                 @RequestParam Integer quantity,
                 Model model) {
     
             Item item = new Item();
             item.setItemName(itemName);
             item.setPrice(price);
             item.setQuantity(quantity);
             itemRepository.save(item);
             model.addAttribute("item", item);
     
             return "basic/item";
         }
     
     //    @PostMapping("/add")    //@ModelAttribute에는 view에서 뿌려질 단어로 쓴다
         public String addItemV2(@ModelAttribute("item") Item item, Model model) {
     //        Item item = new Item();
     //        item.setItemName(itemName);
     //        item.setPrice(price);
     //        item.setQuantity(quantity);
             itemRepository.save(item);
     //        model.addAttribute("item", item); 자동추가, 생략가능
             return "basic/item";
         }
     
     //    @PostMapping("/add")    //가져오는 변수명 생략가능(클래스명 가져옴)
         public String addItemV3(@ModelAttribute Item item) {
             itemRepository.save(item);
             return "basic/item";
         }
     
         @PostMapping("/add")    //아예 @ModelAttribute도 생략가능
                                    //대상객체가 자동으로 모델에 등록된다
         public String addItemV4(Item item) {
             itemRepository.save(item);
             return "basic/item";
         }
     ```

     

7. PRG Post/Redirect/Get

   - redirect하면 get/items/{id}로 다시 서버에 요청함

   - 새로고침해도 다시 화면출력요청을 할 뿐이니 데이터에 변화가 있지않음

     ![image](https://user-images.githubusercontent.com/68681443/132298472-6c50197f-f62e-4682-9fbc-b61b3bb100e9.png)

   - 코드는 아래와 같이 사용할 수 있다

     ```java
     @PostMapping("/add")
         public String addItemV5(Item item) {
             itemRepository.save(item);
             return "redirect:/basic/items/" + item.getId();
         }
     
     @PostMapping("/{itemId}/edit")
         public String edit(@PathVariable Long itemId, @ModelAttribute Item item){
             itemRepository.update(itemId, item);
             return "redirect:/basic/items/{itemId}";    
             								//PathVariable에 들어오는 값을 redirect에서도 쓸수있다
         }
     ```

     

8. RedirectAttributes

   - redirectAttributes에도 addAttribute가능! 
   - return에 들어가는 변수와 다른 status같은것들은 ?status=true 식으로 뒤에 쿼리파라미터로 붙는다

   ```java
   @PostMapping("/add")
       public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
           Item savedItem = itemRepository.save(item);
           redirectAttributes.addAttribute("itemId", savedItem.getId());
           redirectAttributes.addAttribute("status", true);
           return "redirect:/basic/items/{itemId}";
       }
   ```

   ```html
   <h2 th:if="${param.status}" th:text="'저장 완료'"></h2>
   ```

   - redirectAttributes.addAttribute로 파라미터 추가시, param.status로 받을 수 있다

