# 특정 Node의 맨 마지막 글자 뒤에 포커스 주기

## 들어가며
태그 기능을 구현하다 태그 입력을 완료하면 박싱을 하고, 현재 작성중인 태그를 전부 지우면 이전의 태그로 포커스를 옮겨서 수정을 할 수 있는 ui를 구현하기위해
특정 DOM Element에 마지막 글자 뒤에 포커스를 주는 코드의 스니핏을 찾다가 한 코드뭉치를 찾아 적용했다.

이 과정에서 이해안되는 부분도 있었고 코드도 정리할 겸 레퍼런스를 확인해가며 이 코드 스니핏을 개선해보려 한다.
구글링해서 찾은 코드 스니핏은 [스택 오버플로우](https://stackoverflow.com/questions/28270302/jquery-how-to-focus-on-last-character-of-div)의 답변에서 찾았다.

```javascript
function placeCaretAtEnd(el) {
    el.focus();
    if (typeof window.getSelection != "undefined"
            && typeof document.createRange != "undefined") {
        var range = document.createRange();
        range.selectNodeContents(el);
        range.collapse(false);
        var sel = window.getSelection();
        sel.removeAllRanges();
        sel.addRange(range);
    } else if (typeof document.body.createTextRange != "undefined") {
        var textRange = document.body.createTextRange();
        textRange.moveToElementText(el);
        textRange.collapse(false);
        textRange.select();
    }
}
```

하나씩 뜯어보도록 하자.

## Selection API
Selection API는 사용자나 작성자가 문서의 일부를 선택하거나 복사와 붙여넣기를 비롯한 편집 작업을 위한 특정 지점을 지정할 수 있게 한다.

IE9 and Firefox 6.0a2 allow arbitrary ranges in the selection, which follows what this spec originally said
IS9 와 Firefox 6.0a2 는 Selection 에서 임의의 범위를 허용하며, 이는 이 스펙이 원래 말한 내용을 따르고 있다.

However, this leads to unpleasant corner cases that authors, implementers, and spec writers all have to deal with, and they don't make any real sense

하지만 이것은 작성자, 구현자, 그리고 스펙 작성자들이 모두 다루어야 하는 불쾌한 예외 케이스로 이어지며, 현실적으로 불가능하다.

Chrome 14 dev and Opera 11.11 aggressively normalize selections, like not letting them lie inside empty elements and things like that, but this is also viewed as a bad idea, because it takes flexibility away from authors.

Chrome 14 dev 와 Opera 11.11 은 빈 요소와 같은 것들 안에서는 허용하지 않도록 Selection 을 적극적으로 표준화 하였지만, 이 또한 작성자의 유연성을 빼앗기 때문에 나쁜 아이디어라고 할 수 있다.

So I changed the spec to a made-up compromise that allows some simplification but doesn't constrain authors much.

그래서 나는 변경했다 스펙을 꾸며낸 타협으로 단순화를 허용하지만 작성자를 많이 제한하지 않는 방향으로

관련된 논의를 보아라.. 뭐 웹킷/오페라의 행동을 모질라/IE 로 맞추는게 낫겠다 이런 내용

Basically it would throw exceptions in some places to try to stop the selection from containing a range that had a boundary point other than an Element or Text node, or a boundary point that didn't descend from a Document.

기본적으로 Element나 Text Node 이외의 경계점 또는 문서에서 내려오지 않는 경계점을 포함하는 범위를 선택하지 않도록 일부 장소에 예외를 던질 수 있다.

But this meant getRangeAt() had to start returning a copy, not a reference

그러나 이것은 `getRangeAt()` 가 반드시 참조가 아닌 복사본을 반환하기 시작해야 한다는것을 의미한다.

Also, it would be prone to things failing weirdly in corner cases

또한 이것은 코너케이스에서 이상하게 실패하는 경향이 있을 것이다.

prone - 경향이 있는, 실수 하기 쉽다.
weirdly - 불가사의하게 이상하게

Perhaps most significantly, all sorts of problems might arise when DOM mutations transpire, like if a boundary point's node is removed from its parent and the mutation rules would place the new boundary point inside a non-Text/Element node




# window.getSelection
