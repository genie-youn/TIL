# 들어가며
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

# Selection API
Selection API는 사용자나 작성자가 문서의 일부를 선택하거나 복사와 붙여넣기를 비롯한 편집 작업을 위한 특정 지점을 지정할 수 있게 한다.
