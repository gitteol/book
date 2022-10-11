# 새 블록 추가하기

깃털에 새 블록을 추가하는 방법에 대해서 설명합니다.

## Entry JS에서 블록 정보 확인하기

깃털은 [Entry JS](https://github.com/entrylabs/entryjs)와 동일한 동작을 하는 것을 목표로 하고 있습니다.
따라서 블록을 개발하기 전 Entry JS 구현을 확인해 보는 것이 좋습니다.

Entry JS에서 블록 명세는 [`src/playground/blocks`](https://github.com/entrylabs/entryjs/tree/develop/src/playground/blocks) 하위에 위치해 있습니다. 이 곳에서 구현하고자 하는 블록의 정보를 찾아보세요.

예를 들어 `x 좌표를 10 만큼 바꾸기` 블록의 경우 [`src/playground/blocks/block_moving.js`](https://github.com/entrylabs/entryjs/blob/25d493f2848a355ef7f5ceef347a6b5a207f40af/src/playground/blocks/block_moving.js#L345-L397) 파일에 아래와 같이 위치해 있습니다.

```js
move_x: {
    color: EntryStatic.colorSet.block.default.MOVING,
    outerLine: EntryStatic.colorSet.block.darken.MOVING,
    skeleton: 'basic',
    statements: [],
    // ...
    func(sprite, script) {
        const value = script.getNumberValue('VALUE', script);
        sprite.setX(sprite.getX() + value);
        if (sprite.brush && !sprite.brush.stop) {
            sprite.brush.lineTo(sprite.getX(), sprite.getY() * -1);
        }
        return script.callReturn();
    },
    syntax: { js: [], py: ['Entry.add_x(%1)'] },
},
```

여기서 중요한 것은 1번 줄의 `move_x`와 7번 줄의 `func` 함수입니다.

`move_x`는 Entry JS 내부에서 사용하는 블록의 명칭으로 깃털에서도 같은 명칭을 사용합니다.

`func` 함수는 블록의 실제 로직을 담당하는 함수입니다. 이 함수의 로직을 참고해 개발하시면 됩니다.

> Entry JS의 블록 명세에 대해 더 자세히 알고 싶으신 분들은
> [엔트리 개발 가이드](https://docs.playentry.org/guide/index.html)에서
> `Entry JS/블록 명세 작성`과 `Entry JS/블록 모양별 개발 방법` 문서를 참고하세요.

## 블록 개발하기

## 블록 등록하기
