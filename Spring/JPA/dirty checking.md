# Dirty Checking

> transaction 안에서 Entity의 변화를 스스로 감지해 commit시에 자동으로 DB에 반영되는 것을 의미한다.

----

## JPA에서의 데이터 수정과 Dirty Checking



Entity 수정시에 dirty checking에 대해서 알아놓는 것은 필수이다.
우리는 jpa를 사용해서 entity를 수정하고하 할 때 총 2가지 방법이 존재하지만, 사실 merge 방법이 가지는 치명적인 단점(밑에서 알아보자)이 존재해서 dirty checking인 "변경감지"를 사용해야한다.

### dirty checking의 예시를 먼저 살펴보자.


- controller code

```
@PostMapping("items/{itemId}/edit")
public String updateItem(@PathVariable Long itemId, @ModelAttribute("form") BookForm form){

    itemService.updateItem(itemId, form.getName(), form.getPrice(), form.getStockQuantity());

    return "redirect:/items";

}
```

- service code

```
@Transactional
public Item updateItem(Long itemId, String name, int price, int stockQuantity){
    Item findItem = itemRepository.findOne(itemId);
    //findItem에 영속성이 생김 -> 변경 감지를 통해 자동으로 commit된후 flush되어 적용됨
    findItem.setPrice(price);
    findItem.setName(name);
    findItem.setStockQuantity(stockQuantity);

    return findItem;
}
```

(사실 `DTO`를 사용하는 것이 더 좋겠지만 예제니까 간단히 하도록 한다.<br>)


위의 코드를 보면 method name으로 update하는 기능인 것을 알 수 있을 것이다. 그러나 `.save()`와 같은 코드를 찾을 수 없다. 그런데 어떻게 update를 한다는 것인가?에 대해서 의문이 생길 것이다. <br>
가능한 이유는 바로 **transaction과 영속성** 때문이다.<br>
그렇다면 transactioin과 영속성에 대해서 알아보자.<br>

-----


- transaction과 영속성

    > 영속성이란 trasaction단위에서 entity를 프로그램의 실행이 종료되더라도 사라지지 않는 특성을 의미한다.<br>

    `EntityManager`를 통해서 transaction 단위안에서 Entity 저장, 조회, 삭제, 수정 등을 관리할 수 있다.<br>
    중요한 특징은, 알아서 변화된 부분에 대해서 감지를 해주고 (set을 통해서 값을 update하면 알아서 감지) transaction commit이 발생하면, 자동으로 변화된 부분만 update를 해준다는 것이다.<br>
    한다디로, Dirty Checking을 제공한다는 것이다.<br>


다시 예제로 돌아와서 살펴보면, <br>
Dirty checking이 `@Transactional`이 적혀있어서 entity의 영속성을 사용하기만 하여 데이터값을 변경하면 자동으로 일어난다는 것을 알수있다.<br>

이때, `findOne`을 사용해서 준영속상태의 entity에 영속성을 부여하는 과정을 거쳐서 fineItem에 영속성이 생기게 만드는 작업이 중요하다.<br>
그렇다면 영속성을 부여하는 과정이 왜 필요한지에 대해서부터 알아야한다.

-----

## 준영속 객체에 영속성을 부여하기

update를 하는 과정은 보통 id를 parameter로 전달받아서 변경하고하자는 객체를 id로 접근해서 값을 변경하는 로직으로 처리될 것이다. <br>
이 과정에서 객체를 id로 접근하게 되면 **이미 데이터베이스에 있던 데이터를 가지고 오는 것이기에 준영속 객체**가 된다.<br>
준영속 객체는 영속성이 보장해주지 않기에 영속성을 따로 부여해줘야한다.<br>
그렇다는 것은 `EntityManager`를 통해서 id값으로 해당하는 객체를 꺼내오는 작업을 해주면 된다는 것을 의미한다.<br>

- EntityManager를 활용해서 기존의 객체를 불러와서 사용하기

----

다시 예제로 돌아오자, 그렇다면 이제 영속성을 부여했으니 변화시킬 값만 setter를 통해서 바꿔주면 된다.<br>
그렇다면, 자동으로 변경감지가 일어나서 commit될 때 변경된 부분만 update될 것이다.<br>

사실 여기서 중요한 점이 하나 더 있다!<br>
그것은 그러고보면 영속성을 굳이 부여하지 않고 그냥 id값으로 객체를 불러와서 그 값을 변경해주면 되는게 아닐까? 생각이 들 것이다.<br>

그 과정이 곧 merge를 사용한다는 것인데 이 경우에는 어떻게 될까?를 고민해야한다.<br>


----

## merge의 단점

```
@PostMapping("items/{itemId}/edit")
public String updateItem(@PathVariable Long itemId, @ModelAttribute("form") BookForm form){

        Book book = new Book();
        book.setId(form.getId());//준영속 entity로 영속성 부여를 받지 않음
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setIsbn(form.getIsbn());
        book.setAuthor(form.getAuthor());
        itemService.saveItem(book);

        return "redirect:/items";

    }
```

- saveItem을 호출하면 merge를 하는 로직이 들어있다.<br>

```

public void save(Item item){
    if(item.getId()==null){//new thing
        em.persist(item);
    }else{
        em.merge(item);//이미등록된것 update
    }
}

```

이때, merge는 이미 등록되어있는 것을 하나하나 update해주는 과정이라고 생각하면 된다.<br>
그냥 바꿔치기라고 생각하면 된다.<br>

- merge는 변경시킬 것이 아니라서 null값을 속성으로 넘겼을 때 문제가 발생한다.

    값이 하나하나 다 update 되기 때문에 변경시키고 싶지 않았던 것도 null로 update된다.<br>
    따라서, 기존의 DB에 price값이 30000이였다면 변경하고 싶어서 변경form에 값을 안 넘겼는데도 null이 넘어와서 price값이 null로 될 수 있다.<br>


- 영속성의 dirty checking은 변경된 부분만 바꿔줘서 null인 것은 변경된게 아니라고 알아서 판단해줘서 위의 문제를 해결할 수 있다.<br>

<br>
따라서, 우리는 수정을 하고 싶다라면 merge가 아닌 영속성을 활용한 변경감지를 사용해야한다.<br>

-----

## 결론

✍️ 데이터 수정시에는 영속성의 변경감지를 사용한다.

